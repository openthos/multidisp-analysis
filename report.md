# 12.25

## 毕设题目
Android系统多屏幕异构应用显示技术研究

##毕设目标
###底线目标
####开题前：
完成在PC上支持在两个显示器上现实Android的apps.

0. 学习wiki文档（https://github.com/openthos/openthos/wiki）
1. 在配发的笔记本的qemu+kvm模式下，运行Android-x86-64 Lollipop.(包钧圳已完成，直接去实验室请教他)
2. 在配发的笔记本中的ubuntu 15.10下，下载编译Android-x86-64 Lollipop（直接到实验室去下载本地Mirror，直接请教刘浩）
3. 在PC或笔记本上（只有Intel集成显卡），分析配置修改Android-x86-64 Lollipop，支持在两个显示器上现实Android的apps.

参考信息：

1. http://developer.android.com/intl/zh-cn/about/versions/android-4.2.html#SecondaryDisplays
2. http://www.e-consystems.com/blog/linux-android/?p=637
2. http://www.lnttechservices.com/media/31032/wp_eid.pdf
3. http://www.cnx-software.com/2015/01/20/independent-dual-display-support-on-firefly-rk3288-development-board-video/
4. https://boundarydevices.com/android-lollipop-dual-display-imx6/
5. https://github.com/boundarydevices/OkienkaTest

#### 中期检查前：
使用lxc技术，将linux系统（比如fedora或者ubuntu）运行在android x86上  

>> chyyuu: 这个同方的人都安装过了，没有问题，如果前面的工作完成的快，这只是一个照着安装的过程。先完成开题前工作，中期的细节需要我们后续再讨论

底线目标的困难：  
（1）lxc在android的安装  
这篇文章最后提到了LXC on Android devices可能要稍微改改android的内核…… 感觉不太靠谱。 https://www.stgraber.org/2013/12/23/lxc-1-0-some-more-advanced-container-usage/  
继续寻找其他的说法  
这个人好像做了类似的事情，我仔细看一下。https://droidconin.talkfunnel.com/2013/889-lxc-on-android-running-ics-and-jellybean-simultane  
好像也挺麻烦的  
他给了一个demo视频：https://www.youtube.com/watch?v=UpIFByNLM5U  
下面也有人问他lxc在android上的setup，他表示这个setup不太稳定…………有问题发他email：kumarsukhani@gmail.com，到时候不行我就试试看  
另外还有个他的演讲视频：https://www.youtube.com/watch?v=noGwz1mw1aM  
（2）android x86的编译  
我猜我们应该安装android 6.0吧……一是比较新，而是6.0实验性地支持了多窗口（虽然不知道有什么用）  
哈哈看到有人吐槽说：”警告：目前这是个实验性功能，可能导致系统不稳定甚至崩溃，使用者责任自负！！！“  
然后android x86项目的主页在这里：http://www.android-x86.org/  
这个主页表示还没有android 6.0 x86的release版，只有源码，所以要编译巨大的源码……于是一个问题是能不能编的过另一个问题是编不过怎么办……（不知道组里有没有正在做这件事呢？）  
主要问题应该是第一个……可是先要做的却是第二个  

###进阶目标

将linux显示在不同于android的屏幕上  

进阶目标的困难：  
（1）简单来讲就是android本身对于多屏幕的支持  
http://stackoverflow.com/questions/30899716/how-do-i-configure-android-x86-for-dual-monitor-i-e-cloning  
https://www.youtube.com/watch?v=q0G5t0DXMM0  
感觉，把android指向另一个屏幕已经是极限了的感觉…至少对于当时的版本。不过我感觉不太可能吧，再看看？  
https://www.youtube.com/watch?v=ms6KrtIn5ro 这个人实现了使用两个屏幕，不过没有详细的信息  
http://developer.android.com/guide/practices/screens_support.html 这个android的官方文档好像有说怎么做，等有时间仔细看下  

###更高目标

“android已经支持3D，要求linux也支持3D” 这是陈老师的原话，其实我还没太懂  
啥叫linux不支持3d？我再问问陈老师吧！  

>> chyyuu: 等你对3D支持有一定了解后，我们再讨论

##相关论文




