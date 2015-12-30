# 基于markdown格式的开题报告ppt
---

## 毕业设计题目
Android x86下多OS显示技术研究

>（之前商定的题目我感觉有点不妥。我认为我的研究重点是在Android x86下，两个不同的操作系统如何协调显示在不同的屏幕上，并且合理分配键盘鼠标资源。擅自更名，还请陈老师指教。）

> 同意，另外，只要用到mesa，就直接支持3D了。而你的研究中，mesa是系统运行的必经之路，所以你做的自然支持3D

## 研究背景与意义
Android系统作为当下最流行的移动设备操作系统之一，功能也变得日益强大。Android x86作为Android系统的一支，拥有着相当的发展潜力。然而，目前Android x86下应用的开发，还是要使用另外一台装有其他主流桌面操作系统设备搭建开发环境，测试成功后再传给Android系统。这样会导致很多麻烦。一个直接的想法是将某桌面操作系统（比如Fedora 23）运行在Android x86上，这样可以在同一台系统上，开发和使用Android x86的应用。

因此，我希望进行将Linux运行在Android x86系统的研究。我计划使用lxc/chroot技术。这是因为对比虚拟机，chroot高效且节省资源。同时，为了更好的用户体验，我计划将Android x86和Linux分别显示在不同的屏幕上，并且合理分配鼠标，键盘等输入输出资源。

## 研究现状
Columbia University曾经设计出一种基于Linux Container的，在一台移动设备上同时运行多个Android系统的解决方案，叫做Cell[^Cell]。Linux Container的基本思路是只用同一个Kernel来运行多个系统。它可以成功运行多个版本的Android在Nexus 1 和Nexus S 上面。同时，浙江大学的Condroid[^Condroid]也做了类似的事情，同样使用了lxc技术。对于lxc技术和传统的VM的比较，IBM的这篇文章[^Comparison]进行了很详尽的讨论，结论是在大多数的情况下，lxc技术比传统虚拟机技术的表现要好，或者至少持平。这里[^lxc]对lxc有一个不错的讲述。

另一种解决方案是同时运行多个Kernel，比如Popcorn[^Popcorn]。这种方法在boot的时候同时启动多个kernel。这种方法比较复杂，暂不考虑。

无论是Cell还是Condroid都是在移动设备上实现的，并没有实现在Android X86上面。同时，二者也没有考虑多屏幕的支持问题。


## 研究内容方案
### 研究目标
实现利用lxc技术（或者chroot技术 in android）将Linux的发行版（Fedora 23 which using wayland）运行在Android_x86_64_marshmallow环境下。要求实现linux和Android显示在两个不同屏幕上，并且能够合理控制分配鼠标和键盘等资源。

### 具体方案
1. 完成实验环境配置和相关知识学习
	+ Ubuntu 15.10下，Android X86编译环境配置
	+ Android_X86_84_Marshmallow的编译与安装
	+ 学习Linux Container和Chroot相关知识，了解lxc原理
	+ 学习Android 和Linux 的Graphics subsystem，逐步深入了解Surfaceflinger, Wayland和Mesa
2. 利用LXC或类似技术，将Linux系统（如Fedora 23）运行在android x86上
	+ 在Android x86上安装lxc或类似工具，安装运行Fedora 23
	+ 修改Surfaceflinger/Wayland/Mesa配置或者相关代码，控制Linux显示在第二个屏幕上
3. 管理鼠标键盘等设备
        + 学习和分析理解linux/android的input subsystem
	+ 编写代码，管理鼠标键盘在两个系统之间的切换
4. 测试分析Android/Linux在新环境下与原生环境/QEMU-KVM环境下的2D/3D支持功能和性能分析


## 工作基础
1. 熟悉Linux使用，在Ubuntu下配置Android X86编译环境并实践
2. 双系统安装Android X86，修改grub引导使其正常工作
3. 了解Android多屏幕显示/surfaceflinger原理并尝试
4. 学习Linux Container和Chroot相关知识
5. 了解Android和Linux的绘制流程，熟悉Wayland和Mesa

## 预期成果
实现讲Fedora 23以lxc或类似方式运行在Android_x86_64_marshmallow上，并且显示在额外屏幕之上。同时能够控制键盘鼠标在屏幕之间的切换。

## 进度安排
* 2016.1 - 2016.2 寒假期间，学习lxc，chroot，surfaceflinger, wayland，mesa相关知识
* 2016.3 - 使用lxc技术，将Fedora运行在Android X86
* 2016.4 - 实现双屏显示，开始研究输入设备控制
* 2016.5 - 完成输入设备切换控制，实现2D/3D支持
* 2016.6 - 完成功能和性能对比测试分析，并结题

[^Cell]: http://www.cs.columbia.edu/~cdall/pubs/Cells-SOSP-final.pdf
[^Condroid]: http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7130872
[^lxc]: http://haifux.org/lectures/320/netLec8_final.pdf
[^Comparison]: http://www.cs.nyu.edu/courses/fall14/CSCI-GA.3033-010/vmVcontainers.pdf
[^Popcorn]: http://www.popcornlinux.org/images/publications/barbalace_ols.pdf
