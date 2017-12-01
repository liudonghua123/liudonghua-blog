---
title: Android 5.0 真机使用hierarchyviewer
tags:
  - android
  - hierarchyviewer
id: 645
categories:
  - 学习
date: 2014-12-22 14:03:33
---

hierarchyviewer对于Android开发是一个非常实用方便的工具，可惜不能在真机上（使用官方的BUILDTYPE为user的ROM）使用。<!--more-->

[这里](http://blog.apkudo.com/2012/07/26/enabling-hierarchyviewer-on-rooted-android-devices/)的方法以前也提到过，不过[smali/baksmali ](http://code.google.com/p/smali/)不适用于Android 5.0了（baksmali执行时会提示"services.odex is not an apk, dex file or odex file"错误），所以这里使用另一种方法，修改/default.prop文件，但也不能直接修改，这个文件每次系统重启都会用boot.img中ramdisk的default.prop覆盖，即使root了直接修改，在shell中通过getprop "ro.debuggable"查看也不会生效，所以只能去修改boot.img

<!--more-->

修改的方法/工具在网上有很多，Windows上推荐使用Android Image Kitchen，使用方式也很简单，把boot.img拖拽到unpackimg.bat就可解压，然后修改ramdisk中的default.prop，修改"ro.secure"/"ro.debuggable"(参考[WindowManagerService](https://android.googlesource.com/platform/frameworks/base/+/master/services/core/java/com/android/server/wm/WindowManagerService.java)中的isSystemSecure)值分别如下

[shell]
ro.secure=0
ro.debuggable=1
[/shell]

再次双击运行repackimg.bat就可以生成新的修改过的boot.img（名字是image-new.img），然后adb reboot bootloader重启到bootloader模式，使用fastboot flash boot image-new.img刷入新的boot分区就可以生效，这时hierarchyviewer就可以使用

Linux（以Ubuntu为例），可以使用abootimg这个工具，参考[Android boot.img manipulation](http://k.japko.eu/boot-img-manipulation.html)这篇文章，不过在make时会提示"blkid/blkid.h"头文件找不到，安装libblkid-dev即可

参考资料
1. [Android Image Kitchen -- Unpack/Repack Kernel+Recovery Images, and Edit the ramdisk.](http://forum.xda-developers.com/showthread.php?t=2073775)
2. [Android boot.img manipulation](http://k.japko.eu/boot-img-manipulation.html)
3. [Make your android device True Root!](http://forum.xda-developers.com/showthread.php?t=1794203)
4. [libblkid](http://manpages.ubuntu.com/manpages/trusty/man3/libblkid.3.html)