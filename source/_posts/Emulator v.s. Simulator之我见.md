---
title: Emulator v.s. Simulator之我见
tags:
  - android
  - emulator
  - IOS
  - simulator
id: 71
categories:
  - 学习
date: 2014-10-10 15:07:18
---

最近想尝试一个IOS的应用，但苦于没有真机，就从网上下载了ipa文件，想安装到模拟器上，但查了一些资料显示由于IOS模拟器是intel cpu架构的，而从网上包括app store上下载的ipa是arm架构的，这两种架构下的应用都需要在各自编译环境下编译，不能运行其他架构的应用程序。<!--more-->

后来看到IOS模拟器是叫做IOS Simulator，而之前Android开发的模拟器一般叫做emulator，两个类似的不同单词其实有不同的含义。

以下来自[iEmu：在Linux、Windows、Mac、Android系统上仿真运行iOS应用](http://www.leiphone.com/news/201406/iemu-bring-ios-apps-to-android.html)中的解释
> Emulator n.[计]仿真器。> 
> 
> 通过软件方式，精确地在一种处理器上仿真另一种处理器或者硬件的运行方式。其目的是完全仿真被仿真硬件在接收到各种外界信息的时候的反应。我们现在常见的MAME、ePSXe等都是这一类。> 
> 
> Simulator n.模拟器。> 
> 
> 通过某种手段，来模拟某些东西。不一定要完全正确的原理，追求的只是尽可能的相像。比如DWI、BandJAM等都属于这一类。
我认为简单的说就是emulator相比simulator模拟的更加真实、全面，但这就牺牲了一些性能方面优化的考虑。simulator不用完全都模拟，可以额外做一些优化，这就是我理解的simulator模拟器要比emulator模拟器快的原因。

另外实践中可以看出IOS模拟器相比Android模拟器确实流畅很多，其实x86(intel)架构的Android模拟器（和arm架构的类似也是完全模拟）稍微比arm架构的快一点，Genymotion是Android最快的模拟器，也是x86架构的，但还是没有IOS模拟器启动速度快及流畅。或许苹果以后如果提供arm架构的IOS模拟器，其速度应该介于x86 Android Emulator和x86 IOS Simulator之间吧。