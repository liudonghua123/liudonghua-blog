---
title: 'Android启动顺序/过程详解[译]'
tags:
  - android
  - bootloader
  - init
  - kernel
  - zygote
id: 460
categories:
  - 学习
date: 2014-11-05 14:27:20
---

在[这里](http://www.kpbird.com/2012/11/in-depth-android-boot-sequence-process.html)看到一篇关于Android启动过程的文章，讲的还是不错，为了加深理解和分享这方面的知识，所以就翻译了一下。

当我按了Android手机的电源键发生了什么？
Android启动顺序是什么样的？
什么是Linux内核？
桌面Linux内核和Android内核有什么区别？
什么是bootloader？
什么是Zygote？
什么是x86和ARM Linux？
什么是init.rc？
什么是System Server？

当我在思考Android启动过程时，很多这样的问题萦绕在我脑海里。

我将在这里解释Android系统过程，我希望你能找到以上问题的答案。

[toc]

Android是基于开源的Linux系统，[x86](http://en.wikipedia.org/wiki/X86)（x86是基于Intel 8086 CPU的一系列电脑微处理器指令集架构）是Linux内核使用最多的架构，但是Android设备通常运行在[ARM](http://en.wikipedia.org/wiki/ARM)处理器（ARM是Advanced RISC Machine或之前的Acorn RISC Machine的简称）上，[Intel Xolo](http://xolo.in/xolo-x900-features)设备除外。Xolo装备有Atom 1.6GHz x86处理器。Android启动顺序或嵌入式设备或基于ARM的Linux系统相比桌面Linux有一些区别。在这篇文章里，我将只讲解Android的启动顺序，[Linux启动进程](http://www.ibm.com/developerworks/linux/library/l-linuxboot/)是一篇讲解桌面Linux启动顺序的好文章。

Android设备但你按开机键之后又如下启动阶段
[![Android Boot Squence](http://202.203.209.55:8080/wp-content/uploads/2014/11/Android-Boot-Squence.png)](http://202.203.209.55:8080/wp-content/uploads/2014/11/Android-Boot-Squence.png)

# 阶段1：开机和系统初始化

当开机时，从预先定义好即硬写在ROM里的Boot ROM代码开始执行，加载Bootloader到RAM里，然后开始执行Bootloader

# 阶段2：Bootloader启动加载器

Bootloader是一个很小的程序在启动Android系统之前运行，它是第一个运行的程序，所以会对主板和处理器而专门编写。设备制造商使用一些主流的bootloader例如[redboot](http://ecos.sourceware.org/redboot/),[uboot](http://www.denx.de/wiki/U-Boot), [qi bootloader](http://wiki.openmoko.org/wiki/Qi)或他们自己开发的。这不是Android系统的一部分，通常OEM或运营商可以在这里面添加锁或一些限制

Bootloader的运行有两个阶段，第一阶段：检测外部RAM和加载帮助第二阶段的程序；第二阶段：设置网络，内存等运行内核所必须的环境，bootloader还能提供运行kernel特定的参数或输入来完成特殊任务

Android bootloader代码位于
&lt;Android Source&gt;\bootable\bootloader\legacy\usbloader
legacy loader含有两个主要的文件
1\. init.s 初始化stacks、对BSS段清零，调用main.c中的_main()
2\. main.c 初始化硬件（时钟、主板、键盘、控制台），创建Linux tags

参考以下链接了解更多Android bootloader内容
[https://motorola-global-portal.custhelp.com/app/answers/detail/a_id/86208/~/bootloader-frequently-asked-questions](https://motorola-global-portal.custhelp.com/app/answers/detail/a_id/86208/~/bootloader-frequently-asked-questions)

# 阶段3：内核

Android内核的运行类似于桌面Linux内核，当启动内核时，设置缓存、保留的内存，调度，加载驱动程序。当内核启动完成的第一件事就是找到init程序，然后以root权限运行这个系统进程

# 阶段4：init进程

Init是一个最先运行的进程，我们可以说是根进程或所有其他进程的父进程。Init进程有两个任务，挂载/sys,/dev,/proc文件系统和运行init.rc脚本

init可以在这里找到/system/core/init
inti.rc可以在这里找到/system/core/rootdir/init.rc
readme.txt在这里/system/core/init/readme.txt

Android的init.rc有特殊的格式和规则，他们是Actions、Commands、Services、Options

语法
on &lt;trigger&gt;
&lt;command&gt;
&lt;command&gt;
&lt;command&gt;

Service : Services是由init启动，当退出时可能会重启

service &lt;name&gt; &lt;pathname&gt; [ &lt;argument&gt; ]*
&lt;option&gt;
&lt;option&gt;
...

Options : Options是Service的参数，影响service如何以及怎样运行

让我们看一下默认的init.rc文件，这里我只列出了主要的events和services
[![init.rc_action_service](http://202.203.209.55:8080/wp-content/uploads/2014/11/init.rc_action_service.png)](http://202.203.209.55:8080/wp-content/uploads/2014/11/init.rc_action_service.png)
这个阶段，你可以在屏幕上看到Android的logo

# 阶段5：Zygote and Dalvik

在Java里，我们知道对于每个不同的程序Java虚拟器启动不同的实例来运行，为了让Android应用启动的尽可能快，如果每个程序都使用一个JVM则会消耗很多内存和时间，所以为了克服这个问题，Android系统有一个能在Dalvik虚拟机中共享运行代码，低内存消耗，快速启动的Zygote，Zygote预加载、初始化一些Android SDK或Core Framework的，只读的系统核心库。在虚拟机的每个实例中都有一份核心库和堆对象

Zygote加载过程
1\. 加载[Zygote](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/2.2_r1.1/com/android/internal/os/ZygoteInit.java)类
&lt;Android Source&gt;/frameworks/base/core/java/com/android/internal/os/ZygoteInit.java
2\. registerZygoteSocket() - 为zygote命令连接注册一个server socket
3\. preloadClasses() - "preloaded-classes" 是一个含有需要预加载的类的文本文件，可以在这里找到 &lt;Android Source&gt;/frameworks/base
4\. preloadResources() - preloadResources即原生主题和布局，包括android.R，这些都将通过这个方法加载

这时你可以看到启动动画

# 阶段6：System Service或简称为Services

当完成了以上阶段，系统会请求Zygote启动用原生语言或Java开发的system servers，system servers可以认为是进程，system server作为Android SDK的System Services使用的，system server包含system services

Core Services:

1\. Starting Power Manager
2\. Creating Activity Manager
3\. Starting Telephony Registry
4\. Starting Package Manager
5\. Set Activity Manager Service as System Process
6\. Starting Context Manager
7\. Starting System Context Providers
8\. Starting Battery Service
9\. Starting Alarm Manager
10\. Starting Sensor Service
11\. Starting Window Manager
12\. Starting Bluetooth Service
13\. Starting Mount Service

Other services
1\. Starting Status Bar Service
2\. Starting Hardware Service
3\. Starting NetStat Service
4\. Starting Connectivity Service
5\. Starting Notification Manager
6\. Starting DeviceStorageMonitor Service
7\. Starting Location Manager
8\. Starting Search Service
9\. Starting Clipboard Service
10\. Starting Checkin Service
11\. Starting Wallpaper Service
12\. Starting Audio Service
13\. Starting HeadsetObserver
14\. Starting AdbSettingsObserver

# 阶段7 :  启动完成

一旦System Services在内存中启动后，Android就完成了启动过程，这时"ACTION_BOOT_COMPLETED"标准广播就会触发

&nbsp;
了解相关更多内容可参考
[what is the need of second stage boot loader ? why different bootloaders like first stage and second stage?](http://stackoverflow.com/questions/22455153/what-is-the-need-of-second-stage-boot-loader-why-different-bootloaders-like-fi)
[Android_Booting](http://elinux.org/Android_Booting)
[Android OTA 介紹](https://github.com/jollen/android-ota/blob/master/README.md)
[Android OTA 升级之一：编译升级包](http://blog.csdn.net/zjujoe/article/details/6206010) 系列文章