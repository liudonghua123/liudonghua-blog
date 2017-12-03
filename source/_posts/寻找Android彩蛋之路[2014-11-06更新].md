---
title: '寻找Android彩蛋之路[2014-11-06更新]'
tags:
  - android
  - systemui
id: 290
categories:
  - 学习
date: 2014-10-22 21:04:49
---

本文接着前一篇[Android AVD 折腾记](http://202.203.209.55:8080/?p=276 "Android AVD 折腾记")文章，继续寻找Android彩蛋源码。<!--more-->
之前模拟器出现各种问题之后，就打算直接在真机上root之后启用hierarchyviewer，可做到一半发现/system/framewok/service.odex不存在，虽然有services.jar，但里面是空的，不仅如此之前这里很多odex都没有了，可能是我的刷机包是recovery刷机包刷机时没配置odex优化亦或是Android 5.0默认使用了ART取代Dalvik，不过想想觉得后者可能性比较大，幸好在这里多了一个arm目录，里面就有需要修改的service.odex，那么应该也可以修改吧！先暂时这样猜想，验证之后就知道结论了。不过在一次偶然的机会下，我想截一下之前那个启动Intel 64位模拟器错误信息的图片，然而不可思议的事情发生了，模拟器居然用emulator-x86.exe给启动起来了，你敢信吗，并且最终进入了桌面界面，那叫一个激动啊！首先第一件事就是看一下彩蛋，如下图所示
[![android_eggs_with_hierachyviewer](/resources/2014/10/android_eggs_with_hierachyviewer.png)](/resources/2014/10/android_eggs_with_hierachyviewer.png)
OK，到这里你就无法遁形了，原来你就是com.android.systemui.egg.LLandActivity，就隐藏在systemui里呀，躲得挺深的吗！

<span style="color: #00ccff;">2014-11-03更新</span>
<span style="color: #00ccff;"> 随着Android 5.0的代码以上传到AOSP上，LLandActivity的代码已经可以找到了，这次彩蛋主要就涉及两个Java文件[LLand](https://android.googlesource.com/platform/frameworks/base/+/lollipop-release/packages/SystemUI/src/com/android/systemui/egg/LLand.java)和[LLandActivity](https://android.googlesource.com/platform/frameworks/base/+/lollipop-release/packages/SystemUI/src/com/android/systemui/egg/LLandActivity.java)</span>

<span style="color: #ff00ff;">2014-11-06更新
目前我已从SystemUi中提取出这个小游戏，并且放在[Github](https://github.com/liudonghua123/egg)上，感兴趣的可以看一下！！！</span>

现在知道了这些就可以用一些Android代码在线查找工具找到并阅读源码了，这里顺便总结一下一些常用的在线工具，如果手上有源码，find/grep也可以找到。

1\. [https://android.googlesource.com/ ](https://android.googlesource.com/%20)（官方），可以提供切换分支，查看日志，下载单独的子项目功能，但不提供搜索
[![android_googlesource_com](/resources/2014/10/android_googlesource_com.png)](/resources/2014/10/android_googlesource_com.png)

2\. [https://code.google.com/p/android-source-browsing/source/checkout](https://code.google.com/p/android-source-browsing/source/checkout)，可以很方便的切换子项目、分支。但目前不知怎么了就没更新，目前支持到4.2。
[![android_code_google_android_source_browsing](/resources/2014/10/android_code_google_android_source_browsing.png)](/resources/2014/10/android_code_google_android_source_browsing.png)

3\. [https://github.com/android](https://github.com/android)，在github同步了部分android下的子项目，同步比较勤快。
[![android_github_android](/resources/2014/10/android_github_android.png)](/resources/2014/10/android_github_android.png)

4\. [http://androidxref.com/](http://androidxref.com/)，专业搜索Android源码的第三方网站，目前支持到4.4.4_r1
[![android_xref](/resources/2014/10/android_xref.png)](/resources/2014/10/android_xref.png)

5\. [http://grepcode.com/](http://grepcode.com/)，可提供很多代码的搜索，包括Spring、Hadoop、Eclipse等，当然也包括我们的Android
[![android_grepcode](/resources/2014/10/android_grepcode.png)](/resources/2014/10/android_grepcode.png)

但遗憾的是这里的LLandActivity在以上的代码库里都没有，可能是太新了，还没同步上去，不过Android 5.0的SDK你都发布了，源码怎么还没完全同步呢？
[![android_googlesource_com_systemui](/resources/2014/10/android_googlesource_com_systemui.png)](/resources/2014/10/android_googlesource_com_systemui.png)