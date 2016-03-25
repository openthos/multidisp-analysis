# PerfChecker实现与分析

## 产生背景
在生活中，人们越来越多地接触智能手机。然而，智能手机的软件大多或多或少地存在一些性能缺陷，也就是performance bug。尽管这样的性能缺陷影响人们对软件的使用，人们在解决这种性能缺陷上的进展一直不是很顺利。我们对性能缺陷的理解很少，并且缺乏有效的解决方法。

PerfChecker是在论文“Characterizing and Detecting Performance Bugs for Smartphone Applications”中提出的一款静态分析工具。它可以用来检测智能手机软件能性能缺陷。在这篇论文中，作者分析了常见的智能手机软件性能缺陷的特征，比如性能缺陷的类型以及它们产生的过程。然后针对每一种特征，这篇论文提出了对应的解决方案。最终，整合实现出来的性能缺陷分析工具就是PerfChecker。

## 性能缺陷特征提取
### 性能缺陷类型
在这篇论文中，作者分析了大量市场上的手机应用，从中提取出了销量较大的应用。最终，根据用户反馈的bug report，识别的性能缺陷被分为以下几类：

+ **GUI Lagging / GUI 延迟**

	这是所有所分析的性能缺陷之中，占比例最大的一种类型，占到了75.7%。这种类型的bug非常影响应用程序的反应速度，以及用户体验的平滑性。

+ **Energy Leak / 能量泄露**

	这是占比例第二大的类型，占到了14.3%。由于程序的不当设计或者用户的不当操作，导致只能手机的电量下降速度异常。这也是一种很严重的缺陷。

+ **Memory Bloat / 内存爆炸**

	这种类型的性能缺陷和上一种所占比例相当，占到了11.4%。这种类型的bug可能导致大规模的内存爆炸，从而影响应用程序甚至系统的正常运行。

这三种类型的性能缺陷占到了被观测到的性能缺陷的94.7%。虽然还存在其他类型的性能缺陷，但是出现次数非常少，因此先将其忽略。


### 性能缺陷表现特点
根据分析，我们认为智能手机性能缺陷的表现特点有以下几种：

+ **促发性能缺陷可能仅仅需要很少量的输入数据**

	这是和PC应用程序的性能缺陷相比很大的一点差别。PC应用程序的性能缺陷触发可能需要很大量的数据输入，而智能手机性能缺陷可能由于非常简单的一个输入引发，比如触发一次屏幕事件。

+ **促发性能缺陷可能需要特定的用户操作顺序**

	根据分析，相当一部分的性能缺陷需要用户以特定的序列操作才能引发。这个对于开发者重现bug造成了相当大的困难。开发者经常无法预料到用户的特定操作序列，因此难以进行有效的debug操作。

+ **缺乏自动化的衡量策略**

	我们衡量一个问题是否是一个性能缺陷bug没有一种很通用的自动化衡量标准。我们可能依赖以下几点来判断性能缺陷：

	- 人类报告
	- 同类产品比较
	- 开发者的共识

	因此，在衡量一个bug是否是性能bug的时候，难免会占用很多复杂的人类劳动。
	
+ **性能缺陷可能依赖于实际使用平台**

	一个性能缺陷可能在某一个平台上出现，而再另一个平台上不出现。由于智能手机平台的多样性，导致只能手机的性能缺陷的发现与解决存在很大的困难。

综上所述，无论是诊断还是修复一个性能缺陷，尤其是在智能手机的应用里面的性能缺陷，都存在很大的困难。有数据表明，智能手机的性能缺陷和非性能缺陷的bug相比，修复时间更长，评论更多，并且解决bug需要的patch更大。

### 常见的智能手机性能缺陷模式
对于上述讨论的性能缺陷，分析其原因，发现有相当一大部分的性能缺陷都是如下几种模式导致的：

+ **主线程中的复杂操作**

	由于在android之中，所有的GUI操作都是在主线程之中完成的。因此，如果在主线程里面进行复杂的操作，那么势必会导致GUI的卡顿，即产生性能缺陷。常见的复杂操作可能会涉及数据库查询，网络访问等等。

+ **隐身GUI的计算**

	当一个android程序被放置在后台，它仍然可能在后台继续执行。比如当gps返回当前位置变化的时候，可能会引发GUI的变化。然而这部分的变化是不需要立即计算的。这些更新可以在这个应用程序被放回前台的时候一并计算。

+ **经常使用的复杂回调**

	在实际开发中，开发者经常会写出一些病态的、效率低下的回调函数。比如list view的回调函数。当开发者自行设计一种listitem的时候，需要实现getview函数。当一个item由于list的滚动导致出现在屏幕上的时候，系统会调用这个getview函数。在这个函数中需要访问并且生成该item的layout（这个过程比较慢）。因此在android开发中，开发者应该使用系统提供的recycledview和view holder模式来加速getview的过程。如果开发者没有这么做，则可能造成卡顿现象的出现。

## 性能缺陷检测的具体实现过程
PerfChecker可以检测两种上述提到的性能缺陷：**主线程中的复杂操作** 和 **view holder模式的使用检测**

在实际操作上，PerfChecker输入的是智能手机应用的.class文件，然后使用java的静态分析库Soot来进行静态分析。然后进行上述两种性能缺陷的检测。下面具体讲解实现过程。

### 主线程中的复杂操作的检测
1. 扫描寻找需要分析的类

	主线程中的复杂操作可能来自于两个地方：**Activity的生命循环** 和 **GUI事件的回调事件**
	
	因此，针对这两个来源，我们使用不同的检测方法。

	a. **Activity的生命循环的检测**
	
		首先，我们遍历所有的class。对于每一个class，迭代判断他的父类。如果其包含android.app.Activity字段，则认为这个类是继承Activity的类，因此需要分析。
		
		```
		if (checkLifecycle != 0) {
			while (tmp.getSuperclass() != null) {
				if (tmp.getName().contains("android.app.Activity")) {
					toAnalyse = true;
				}
    		tmp = tmp.getSuperclass();
			}
		}
		```		

	b. **GUI事件的回调事件检测**
	
		同样，我们遍历所有的类。然后我们研究发现，所有包含GUI事件回调函数的类，都会被编译成XXXXX$YYYYY.class的形式。因此我们找出所有这种形式的类，观察弃是否真的包含GUI回调函数。为此，我们事先准备好了一系列GUI回调函数的列表（见附录ListenerList）。得到并且遍历改类的所有接口，如果存在上面提到的列表里面的函数，则认为这个类需要分析。
		
	
		```
		if (checkGUI != 0 && className.contains("$")) {
			for (Class clInterface : cl.getInterfaces()) {
				if (ListenerList.list.contains(clInterface.getName())) {
					toAnalyse = true;
				}
			}
		} 
		```
2. 找到该类的entrypoint

	上一步中，我们得到了所有需要分析的类。接下来，我们需要找到这些类可能的入口。我们假定，对于一个函数，如果它以on作为前缀，则他很可能是一个入口方法。
	
	```
	List<SootMethod> methods = sootClass.getMethods();
	ArrayList<SootMethod> entrypoints = new ArrayList<SootMethod>();
	Iterator<SootMethod> itMethods = methods.iterator();
	while (itMethods.hasNext()) {
		SootMethod method = itMethods.next();
		if (method.getName().startsWith("on")) {
			entrypoints.add(method);
		}
	}
    ```

3. 得到entrypoint的callgraph，并且得到从这个函数入口进入，所有可能到达的目标函数。
	
	```
	CallGraph CG = Scene.v().getCallGraph();
    ArrayList<SootMethod> listMe = new ArrayList<SootMethod>();
    listMe.add(method);
    ReachableMethods rm = new ReachableMethods(CG, (Collection)listMe);
    ```
4. 判断目标函数集合中是否存在复杂操作。

	我们事先存好一系列复杂操作的函数签名，比如数据库访问、网络通信等等（见附录LengthyOprList）。然后遍历上面得到的目标函数是否属于这个List。若属于，则记录到result中。
	
	```
	if (LengthyOprList.list.contains(tgtMethod.getSignature())) {
		result.addResult(sootClass, method, "MAY CALL " + tgtMethod.getSignature());
	}
	```

### view holder模式的使用检测
1. 扫描寻找需要分析的类

	我们的目的是找到所有开发者编写的getView函数，因此我们去找需要实现这个函数的类的特征。我们发现，我们所有需要实现这个函数的类都是android.widget.BaseAdapter、android.widget.ListAdapter、android.widget.SpinnerAdapter之一的子类。因此我们类似上面的方法，迭代寻找被测试类的父类，判断其是不是这三个类之一。如果是，则认为这个类需要分析。
	
	```
	while (tmp.getSuperclass() != null) {
		if (tmp.getName().contains("android.widget.BaseAdapter") 
			|| tmp.getName().contains("android.widget.ListAdapter")
			|| tmp.getName().contains("android.widget.SpinnerAdapter")) {
			toAnalyse = true;
		}
		tmp = tmp.getSuperclass();
	}
	```
	
2. 找到该类的getView函数，并且得到程序依赖图(Program Dependency Graph)的对应部分

	我们可以使用Soot提供的API来找到一个函数在程序依赖图中对应的部分。
	
	```
	SootMethod me = cl.getMethod("android.view.View getView(int,android.view.View,android.view.ViewGroup)");
	ExceptionalUnitGraph EUG = new ExceptionalUnitGraph(me.retrieveActiveBody());
	HashMutablePDG PDG = new HashMutablePDG(EUG);
	```
	
3. 分析该程序依赖图，判断是否使用了recycledview进行加速

	程序依赖图中的节点分为两种，一种叫做CFGNode，也就是控制程序流的节点，比如if判断；另一种是Region，这个代表其他的程序语句。因此，为了判断开发者是否依据recycledview是否存在来编写getView函数，我们需要先找到PDG中所有的CFGNode。由于recycledview是函数的第二个参数，因此如果开发者判断了recycledview是否存在，则一定会出现"r1 == null"或者"r1 != null"的字样。我们根据这个来判断开发者是否使用了viewholder模式
	
	```
	if (node.getType() == PDGNode.Type.CFGNODE) {
		if (node.toString().contains("r1 == null") || node.toString().contains("r1 != null")) {
			checked = true;
		}
	}
	```
	
## PerfChecker运行结果
我们使用的被测试的应用程序为Omnidroid。这是一个NYU开发的一个android的事件管理工具。该应用程序不大，并且依赖的android10.jar也不大，因此很适合作为测试工具。

我们从三个方面测试该应用的Performance Bug：Activity生命周期中的复杂操作、GUI事件回调函数中的复杂操作、viewholder模式使用检测。

### 运行结果

#### Activity生命周期中的复杂操作检测
测试结果发现有6个类的入口可能会调用复杂操作。这些复杂操作均为数据库访问。

```
Printing Results
edu.nyu.cs.omnidroid.app.view.simple.ActivityChooseFiltersAndActions
		void onActivityResult
				MAY CALL android.database.Cursor query
		boolean onContextItemSelected
				MAY CALL android.database.Cursor query
edu.nyu.cs.omnidroid.app.view.simple.ActivityDlgApplicationLoginInput
		void onCreate
				MAY CALL android.database.Cursor query
edu.nyu.cs.omnidroid.app.view.simple.ActivityDlgLog
		void onCreate
				MAY CALL android.database.Cursor query
				MAY CALL android.database.Cursor query
edu.nyu.cs.omnidroid.app.view.simple.ActivityLogTabs
		void onCreate
				MAY CALL android.database.Cursor query
		void onResume
				MAY CALL android.database.Cursor query
		boolean onOptionsItemSelected
				MAY CALL android.database.Cursor query
edu.nyu.cs.omnidroid.app.view.simple.ActivityMain
		void onCreate
				MAY CALL android.database.Cursor query
		void onResume
				MAY CALL android.database.Cursor query
edu.nyu.cs.omnidroid.app.view.simple.ActivitySavedRules
		void onCreate
				MAY CALL android.database.Cursor query
		void onActivityResult
				MAY CALL android.database.Cursor query
		boolean onContextItemSelected
				MAY CALL android.database.Cursor query
				MAY CALL android.database.Cursor query
		boolean onOptionsItemSelected
				MAY CALL android.database.Cursor query
```

#### GUI事件回调函数中的复杂操作检测
测试结果发现有2个类的入口可能会调用复杂操作。这些复杂操作同样均为数据库访问。

```
Printing Results
edu.nyu.cs.omnidroid.app.view.simple.ActivitySavedRules$1
		void onItemClick
				MAY CALL android.database.Cursor query
edu.nyu.cs.omnidroid.app.view.simple.ActivitySavedRules$RuleListAdapter$1
		void onClick
				MAY CALL android.database.Cursor query
```

#### viewholder模式使用检测
测试结果发现有9个getView函数没有使用给定的recycledView，也就是没有利用ViewHolder模式。

```
Printing Results
edu.nyu.cs.omnidroid.app.view.simple.ActivityChooseRootEvent$AdapterEvents
		android.view.View getView
				Recycled View Not Used
edu.nyu.cs.omnidroid.app.view.simple.ActivityDlgActionInput$DlgAttributes$AdapterAttributes
		android.view.View getView
				Recycled View Not Used
edu.nyu.cs.omnidroid.app.view.simple.ActivityDlgActions$AdapterActions
		android.view.View getView
				Recycled View Not Used
edu.nyu.cs.omnidroid.app.view.simple.ActivityDlgApplications$AdapterApplications
		android.view.View getView
				Recycled View Not Used
edu.nyu.cs.omnidroid.app.view.simple.ActivityDlgAttributes$AdapterAttributes
		android.view.View getView
				Recycled View Not Used
edu.nyu.cs.omnidroid.app.view.simple.ActivityDlgFilters$AdapterFilters
		android.view.View getView
				Recycled View Not Used
edu.nyu.cs.omnidroid.app.view.simple.ActivityLogTabs$LogAdapter
		android.view.View getView
				Recycled View Not Used
edu.nyu.cs.omnidroid.app.view.simple.ActivitySavedRules$RuleListAdapter
		android.view.View getView
				Recycled View Not Used
edu.nyu.cs.omnidroid.app.view.simple.AdapterRule
		android.view.View getView
				Recycled View Not Used
```

### 结果分析
可以看出，PerfChecker确实能像论文中声称的，找出特定类型的潜在性能缺陷。这些分析结果对于程序员调试性能缺陷bug肯定会有一定的帮助。

然而，我认为，PerfChecker有如下的缺点：

1. PerfChecker的原理是基于静态分析，因此就会存在普遍的静态分析可能导致的各种问题。静态分析知识从代码上来分析程序，没有实际将程序运行起来，存在相当的局限性。比如，程序真正运行起来的时候，实际的控制流和数据流，静态分析是无法获得的。例如，程序出现性能bug的通路是无法获得的。然而我们并没有去除正常运行时程序经过的通路。
2. PerfChecker给出的结果，可能确实是在动态分析中发生了，可是我们无法判断出这是否造成了性能bug。比如在主线程中，我们访问了数据量很小的数据库，这个对性能并不会有什么影响，然而PerfChecker还是会给出警告来提示开发者注意。
3. PerfChecker给出的结果并不能直接转变为解决方法。比如我们可能的确存在在主线程中访问数据库的需求。即使我们将访问数据库这个过程封装在一个新的线程里面，主线程也可能要等待数据库操作执行完毕之后才能继续下一步的工作。因此这种时候这种性能缺陷很难避免。

---
## 附录
### ListenerList
```
android.widget.AbsListView$OnScrollListener
android.widget.AdapterView$OnItemClickListener
android.widget.AdapterView$OnItemLongClickListener
android.widget.AdapterView$OnItemSelectedListener
android.view.View$OnClickListener
android.view.View$OnLongClickListener
android.view.View$OnFocusChangeListener
android.view.View$OnKeyListener
android.view.View$OnTouchListener
android.view.View$OnCreateContextMenuListener
android.widget.CalendarView$OnDateChangeListener
android.widget.AutoCompleteTextView$OnDismissListener
android.widget.Chronometer$OnChronometerTickListener
android.widget.CompoundButton$OnCheckedChangeListener
android.widget.DatePicker$OnDateChangedListener
android.widget.ExpandableListView$OnChildClickListener
android.widget.ExpandableListView$OnGroupClickListener
android.widget.ExpandableListView$OnGroupCollapseListener
android.widget.ExpandableListView$OnGroupExpandListener
android.widget.NumberPicker$OnScrollListener
android.widget.NumberPicker$OnValueChangeListener
android.widget.PopupMenu$OnDismissListener
android.widget.PopupMenu$OnMenuItemClickListener
android.widget.PopupWindow$OnDismissListener
android.widget.RadioGroup$OnCheckedChangeListener
android.widget.RatingBar$OnRatingBarChangeListener
android.widget.SearchView$OnCloseListener
android.widget.SearchView$OnQueryTextListener
android.widget.SearchView$OnSuggestionListener
android.widget.SeekBar$OnSeekBarChangeListener
android.widget.ShareActionProvider$OnShareTargetSelectedListener
android.widget.SlidingDrawer$OnDrawerCloseListener
android.widget.SlidingDrawer$OnDrawerOpenListener
android.widget.SlidingDrawer$OnDrawerScrollListener
android.widget.TabHost$OnTabChangeListener
android.widget.TextView$OnEditorActionListener
android.widget.TimePicker$OnTimeChangedListener
android.widget.ZoomButtonsController$OnZoomListener
```

### LengthyOprList
```
java.net.URL, <java.net.URL: java.net.URLConnection openConnection()>
java.net.URL, <java.net.URL: java.io.InputStream openStream()>
java.net.URL, <java.net.URL: java.lang.Object getContent()>
java.net.URL, <java.net.URL: java.lang.Object getContent(java.lang.Class[])>
java.net.URLConnection, <java.net.URLConnection: java.io.InputStream getInputStream()>
java.net.URLConnection, <java.net.URLConnection: java.io.OutputStream getOutputStream()>
java.net.URLConnection, <java.net.URLConnection: java.lang.Object getContent()>
java.net.URLConnection, <java.net.URLConnection: java.lang.Object getContent(java.lang.Class[])>
java.net.URLConnection, <java.net.URLConnection: java.lang.String getContentEncoding()>
java.net.URLConnection, <java.net.URLConnection: java.lang.String getContentType()>
java.net.URLConnection, <java.net.URLConnection: int getContentLength()>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: void execSQL(java.lang.String)>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: void execSQL(java.lang.String,java.lang.Object[])>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: android.database.Cursor query(java.lang.String,java.lang.String[],java.lang.String,java.lang.String[],java.lang.String,java.lang.String,java.lang.String,java.lang.String)>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: android.database.Cursor query(boolean,java.lang.String,java.lang.String[],java.lang.String,java.lang.String[],java.lang.String,java.lang.String,java.lang.String,java.lang.String,android.os.CancellationSignal)>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: android.database.Cursor query(java.lang.String,java.lang.String[],java.lang.String,java.lang.String[],java.lang.String,java.lang.String,java.lang.String)>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: android.database.Cursor query(boolean,java.lang.String,java.lang.String[],java.lang.String,java.lang.String[],java.lang.String,java.lang.String,java.lang.String,java.lang.String)>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: android.database.Cursor queryWithFactory(android.database.sqlite.SQLiteDatabase$CursorFactory,boolean,java.lang.String,java.lang.String[],java.lang.String,java.lang.String[],java.lang.String,java.lang.String,java.lang.String,java.lang.String,android.os.CancellationSignal)>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: android.database.Cursor queryWithFactory(android.database.sqlite.SQLiteDatabase$CursorFactory,boolean,java.lang.String,java.lang.String[],java.lang.String,java.lang.String[],java.lang.String,java.lang.String,java.lang.String,java.lang.String)>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: android.database.Cursor rawQuery(java.lang.String,java.lang.String[],android.os.CancellationSignal)>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: android.database.Cursor rawQuery(java.lang.String,java.lang.String[])>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: android.database.Cursor rawQueryWithFactory(android.database.sqlite.SQLiteDatabase$CursorFactory,java.lang.String,java.lang.String[],java.lang.String)>
android.database.sqlite.SQLiteDatabase, <android.database.sqlite.SQLiteDatabase: android.database.Cursor rawQueryWithFactory(android.database.sqlite.SQLiteDatabase$CursorFactory,java.lang.String,java.lang.String[],java.lang.String,android.os.CancellationSignal)>
android.context.ContextWrapper, <android.context.ContextWrapper: java.io.FileInputStream openFileInput(java.lang.String)>
android.context.ContextWrapper, <android.context.ContextWrapper: java.io.FileOutputStream openFileOutput(java.lang.String,int)>
java.io.Reader, <java.io.Reader: int read()>
java.io.Reader, <java.io.Reader: int read(char[])>
java.io.Reader, <java.io.Reader: int read(java.nio.CharBuffer)>
java.io.InputStreamReader, <java.io.InputStreamReader: int read()>
java.io.InputStreamReader, <java.io.InputStreamReader: int read(char[],int,int)>
java.io.BufferedReader, <java.io.BufferedReader: int read()>
java.io.BufferedReader, <java.io.BufferedReader: int read(char[],int,int)>
java.io.BufferedReader, <java.io.BufferedReader: java.lang.String readLine()>
java.io.Writer, <java.io.Writer: java.io.Writer append(char)>
java.io.Writer, <java.io.Writer: java.io.Writer append(java.lang.CharSequence)>
java.io.Writer, <java.io.Writer: java.io.Writer append(java.lang.CharSequence,int,int)>
java.io.Writer, <java.io.Writer: void write(char[])>
java.io.Writer, <java.io.Writer: void write(int)>
java.io.Writer, <java.io.Writer: void write(java.lang.String)>
java.io.Writer, <java.io.Writer: void write(java.lang.String,int,int)>
java.io.BufferedWriter, <java.io.BufferedWriter: void write(char[],int,int)>
java.io.BufferedWriter, <java.io.BufferedWriter: void write(int)>
java.io.BufferedWriter, <java.io.BufferedWriter: void write(java.lang.String,int,int)>
android.graphics.BitmapFactory, <android.graphics.BitmapFactory: android.graphics.Bitmap decodeFile(java.lang.String,android.graphics.BitmapFactory$Options)>
android.graphics.Bitmap, <android.graphics.Bitmap: boolean compress(android.graphics.Bitmap$CompressFormat,int,java.io.OutputStream)>
```