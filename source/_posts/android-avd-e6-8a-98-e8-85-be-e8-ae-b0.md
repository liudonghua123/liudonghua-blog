---
title: Android AVD 折腾记
tags:
  - android
  - avd
id: 276
categories:
  - 学习
date: 2014-10-22 15:48:07
---

<audio controls autoplay>
    <source src="/resources/2014/10/醉清风.mp3">
</video>
昨天发现Android5.0的彩蛋原来是类似于Flappy Bird的游戏，不过相比更难，如下图，突发奇想想看一下Android 5.0的彩蛋的源码。<!--more-->
正确开启彩蛋的方法是：快速点击"设置-关于手机-Android版本"4/5下，然后会有一个小圆圈，点几下变成大棒棒糖之后，长按即会出现。
[![android_5.0_eggs](/resources/2014/10/android_5.0_eggs.png)](/resources/2014/10/android_5.0_eggs.png)

想到这部分源码不属于GAPPS，所以应该是开源的，查看"recent used apps"(点击底部导航栏最右边虚拟按键)显示这个游戏属于"设置"，所以猜想这块的源码应该在设置里，查看[最新的设置源码](https://android.googlesource.com/platform/packages/apps/Settings/)，最新Tags是android-l-preview_r2，所以相应源码应该就在master分支上，但这么多源码怎么快速找到彩蛋相应的源码呢，试试使用hierarchyviewer可以看到一些View的类名然后再搜索是一个不错的想法，但真机上不能使用hierarchyviewer，只能在模拟器上验证（其实真机也可以只是相对麻烦一点，只需修改WindowManagerService.java中[isSystemSecure](https://github.com/android/platform_frameworks_base/blob/master/services/java/com/android/server/wm/WindowManagerService.java#LC6179)一行代码，返回false即可，不过一般情况下对于普通人修改源码再编译有些困难，不过可以修改编译好的dex文件呀，详情可参考[enabling-hierarchyviewer-on-rooted-android-devices](http://blog.apkudo.com/2012/07/26/enabling-hierarchyviewer-on-rooted-android-devices/)，后面再写一篇文章来简单讲述一下）。

那行，现在就启动模拟器吧，当然首先Intel 64位的模拟器吧，如下图所示
[![android_sdk_manager](/resources/2014/10/android_sdk_manager.png)
](/resources/2014/10/android_sdk_manager.png)不过提示没有VTx支持，如下图所示，进入BIOS开启VTX
[![vt-x-problem](/resources/2014/10/vt-x-problem.png)](/resources/2014/10/vt-x-problem.png)
开启VTx之后安装%ANDROID_HOME%\extras\intel\Hardware_Accelerated_Execution_Manager\intelhaxm.exe，还没开始装又提示如下错误
[![vt-x-not-support](/resources/2014/10/vt-x-not-support.png)](/resources/2014/10/vt-x-not-support.png)
那好吧Google一下还得[关闭Hyper-V](http://stackoverflow.com/questions/16091677/intel-haxm-installation-error-this-computer-does-not-support-intel-virtualizat)，这可以在控制面板-删除或修改程序（Control Panel\Programs\Programs and Features）-打开或关闭Windows特性中关闭之（见下图，去选Hyper-V Platform），或者直接管理员权限运行"dism.exe /Online /Disable-Feature:Microsoft-Hyper-V"即可
[![hyper-v-turn-off](/resources/2014/10/hyper-v-turn-off.png)](/resources/2014/10/hyper-v-turn-off.png)

不过创建Intel x86_64 AVD启动时还是提示"Missing emulator-x86_64"，tools目录下确实没有这个文件，已经是最新的的Android SDK Tools了，这Android团队可真是的，发布了64位模拟器镜像不提供64位模拟器，暂时没办法，如果手上有完整的源码可以编一个出来，现在就使用32位的吧。启动之后直接黑屏，后来参考[这里](http://stackoverflow.com/questions/10022580/android-emulator-shows-nothing-except-black-screen-and-adb-devices-shows-device "android-emulator-shows-nothing-except-black-screen-and-adb-devices-shows-device")修改模拟器设置"Use Host GPU"选中，这时有Android在那一闪一闪了，不过十多分钟过去了，还是没有进展，那就adb log看一下吧，最后循环这个错误，暂时没辙了，那就想办法在真机环境下运行hierarchyviewer吧！
[![avd_startup_problem](/resources/2014/10/avd_startup_problem.png)](/resources/2014/10/avd_startup_problem.png)