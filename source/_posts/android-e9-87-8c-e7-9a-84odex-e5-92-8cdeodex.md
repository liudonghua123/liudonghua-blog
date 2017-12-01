---
title: Android里的Odex和Deodex
tags:
  - android
  - deodex
  - odex
id: 317
categories:
  - 学习
date: 2014-10-23 20:50:25
---

最近看到一篇文章介绍[Odex/Deodex](http://www.addictivetips.com/mobile/what-is-odex-and-deodex-in-android-complete-guide/)，讲的很清楚明白，所以就翻译了一下加深自己的理解。<!--more-->

[toc]

作为一个Android新手，最困扰的就是那些不断出现的超出我理解范围的各种专业术语。由于没有Linux基础背景，所以让我弄懂这些在开发社区广为流传的术语是一件头疼的事。与此同时，由于我不理解这些专业术语，所以我就不知道哪些对我有用哪些对我没用。在我周围，这个问题已经影响到很多初学者甚至是普通用户。

在定制ROM和或内置应用甚至主题时经常出现的一个术语就是deodex和odex，很多用户不理解这些术语真正的含义，而那些开发者还在不断的说他们的主题或ROM是deodex处理过的，这让普通用户更不知道这里面到底做了什么。

在这篇文章里，我们会解释odex和deodex的含义以及这对普通用户来说有什么意义。

###### **什么是ODEX文件**

在Android文件系统中，应用程序是打包成以apk为后缀的文件，这些应用程序或apk文件含有本来用于节省空间的对应的odex文件。这些odex文件其实是应用程序的一部分，在启动之前已经优化过。通过使用预加载一部分应用程序加速了启动进程。另一方面，这无形中让修改这些文件变的困难了，因为一部分代码在运行前已经被分离到其他地方了。

###### **接下来说说DEODEX**

Deodex简单的说是以一种特定的方式——重新装配成一个classes.dex，来打包apk。通过这样做，一个应用程序的所有部分都重新整合在一起，因此再也不用担心修改某个apk会影响到其他一些独立的odex处理过的部分。

总之，Deodex处理过的ROM或apk所有的东西都打包在一起，这允许方便修改，例如修改主题。正因为没有任何代码片段是在外部的，所以自定义的ROM或apk一般都采用deodex处理以保证完整性。

###### **这是如何工作的**

对于很多在我们周围的编程极客，Android系统使用基于Java的叫做Dalvik的虚拟机运行程序。deodex处理过的或直接dex文件包含适用于虚拟机的缓存（即Dalvik-cache），这种缓存存在于apk里。而一个odex的文件等同与一个优化过的dex，并且就存放在apk旁边而不是里面。Android系统里这种机制对于所有的系统应用来说是默认的。

现在当一个基于Android的系统启动时，对于Davlik虚拟机的davlik缓存用这些odex文件提前让一些程序加载，从而加速了启动过程。

通过使用deodex处理这些apk，开发者实际上是吧odex文件重新放回对应的apk里。既然所有的代码现在都包含在apk里，现在就可以修改这个apk而不用担心会和系统运行环境有冲突。

###### **优点和缺点**

使用deodex的优点在于可以方便修改。这广泛用于定制ROM和主题。一个开发者编译一个定制ROM一般总是选择deodex处理ROM里的应用，因为这不仅方便他修改许多apk，而且预留了安装后定制主题的空间。

另一方面，因为odex文件直接支持创建dalvik缓存，删除他们意味着使启动时间变长了，然而这仅对deodex操作后第一次启动时会出现这种情况，因为当程序运行时缓存一样会被创建。如果dalvik缓存由于一些原因被清除了，长时间启动的情况才会出现。

对于一个普通用户，主要都影响在于定制主题。Android系统的主题同样 是在apk里，如果你要修改他们，最好选择deodex版本的ROM。

注：
第一次翻译这类技术文章，显然显得有些生硬，其实很多东西英文是理解的但转换成中文就不是那么简单了。
odex/deodex相关内容还可以参考
[what-are-odex-files-in-android](http://stackoverflow.com/questions/9593527/what-are-odex-files-in-android#answers)
[SCRIPT TO ODEX](http://forum.xda-developers.com/showthread.php?t=2390162)