# 12.29
今天尝试了各种方法都制作不了iso的u盘启动盘（见issue）  
实在没辙了，现在打算make一个efi_img看看有没有帮助  
..
解决办法及后续操作：  
尽管不确定为什么iso今天不能工作，但是忙活了一下午，至少我们让img工作了……  
最后.img编译好之后，dd到usb里面，然后启动的时候选uefi with csm，选择usb即可进入安装。  
吸取上次的教训，不安装grub（为了避免找不到ubuntu还得修复grub）  
安装完成之后，重启发现只能进入ubuntu，设置grub的timeout使得开机时显示grub。同时修改/etc/grub.d里面的40_custom文件，添加android的menuentry。这里提供我调好的一个版本：  
```
menuentry "Android-x86" {
	set root='hd0,msdos3'
	linux /android-2015-12-29/kernel root=/dev/ram0 androidboot.hardware=android_x86_64 DATA= 
	initrd /android-2015-12-29/initrd.img
}
```
注意“_64”的部分，否则图形界面似乎不会出现。   
还要注意要update-grub  
然后重启，就看到grub页面能够选择ubuntu还是android x86了！  



# 12.28
今天对android和linux的绘制流程进行了一定了解  
明天我还是想试试run android x86 on linux in lxc  

# 12.27
管同学要了一个ubuntu liveCD，然后进去按照这个：http://howtoubuntu.org/how-to-repair-restore-reinstall-grub-2-with-a-ubuntu-live-cd 修复了grub  
至少现在ubuntu是回来了，比什么都好……  

趁着现在夜深人静，把6.0下下来吧！  
..  
已成功编译并在marshmallow并在qemu运行，发现要加上-enable-kvm才能正常工作  

# 12.26

下好了，这个难道是因为选了branch所以有iso_img么……先试着make一个userdebug吧，出问题了再说  
好吧出问题了，找不到mako.template，这是啥  
apt-get了这个包之后报了别的问题：curl: (6) could not resolve host:www.broadcom.com  
翻了个墙终于ping通了，重新make  
make completed successfully (01:55:46  
棒！然后看看怎么办……  
用qemu打开之后选择模式（run without installation/ vesa/ debug），但是选哪个都是出现android几个大字之后闪退……是qemu问题还是android问题  
好吧是我自己的问题，现在安卓几个大字开始闪了，敲碗坐等  
5分钟了没什么变化我觉得可以退了……  
..  
..  
研究了一会没研究明白，所以我打算先去找个release版的android x86，要是再有问题应该就是qemu配置不对了吧……  

我把官方release的5.1，用qemu-system-i386 -cdrom xxxxx.iso运行，现象是android几个大字之后闪退。debug mode下也是什么couldn't mount的错误……我觉得我可能qemu的使用方法不对……  
我分别烧进usb试试看吧?  
..  
..  
qemu进去之后按tab修改启动项，添加console=ttyS0之类的没啥效果……  
然后我把官方release的版本写到u盘上，开机可以启动（右键iso第一项就是写到u盘）  
..  
突然就不太行了……改用unetbootin试试  
好奇怪……现在怎么都不能从我u盘启动了……  
..  
..  
..  
终于把android x86装到硬盘上了，但是好像把grub搞崩了，现在开机进不了ubuntu，要怎么修复一下


# 12.25.2
搭建Building Environment，参考http://source.android.com/source/initializing.html  
一步一步来，网速并不快  
开始make才发现搞错了repo……………………………………  
开始重新sync……以后大家一定要注意不要搞错……  
重新sync之后（按照https://github.com/openthos/openthos/wiki/AOSP-6.0-x86_64-kernel-3.10-in-ubuntu-15.04 ）还是不对，bootable下面只有recovery一个文件夹，这样make的时候就找不到iso_img啊……没想清楚是哪里的问题。我问问冯学长。    
另一方面，现在速度好像挺快的，看看能不能直接搞个sf的lollipop/marshmallow下来  



# 12.25.1

## 毕设题目
Android系统多屏幕异构应用显示技术研究

##毕设目标
###底线目标
####开题前：
完成在PC上支持在两个显示器上现实Android的apps.

0. 学习wiki文档（https://github.com/openthos/openthos/wiki）
1. 在配发的笔记本的qemu+kvm模式下，运行Android-x86-64 Lollipop/marshmallow.(包钧圳已完成，直接去实验室请教他)
2. 在配发的笔记本中的ubuntu 15.10下，下载编译Android-x86-64 Lollipop/marshmallow（直接到实验室去下载本地Mirror，直接请教刘浩）
3. 在PC或笔记本上（只有Intel集成显卡），分析配置修改Android-x86-64 Lollipop/marshmallow，支持在两个显示器上现实Android的apps.

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




