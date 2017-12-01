---
title: Windows中JumpList与查看命令行参数
tags:
  - jumplist
  - wmic
id: 357
categories:
  - 学习
date: 2014-10-26 15:44:39
---

在Windows上有个很好的截图软件——winsnap，这个软件截的图默认边缘有立体的阴影效果，看上去很舒服；并且把这个程序Pin到任务栏上，右键应用程序可以看到一个非常快捷的JumpList菜单，可以全屏、应用程序、窗口、对象、区域截图，如下图所示。
[![winsnap_jumplist](http://202.203.209.55:8080/wp-content/uploads/2014/10/winsnap_jumplist-300x291.png)](http://202.203.209.55:8080/wp-content/uploads/2014/10/winsnap_jumplist.png)
<!--more-->
但昨天装了新版本的官方最新版本的winsnap，右键发现没有了JumpList功能，我比较常用的区域截图没有jumplist快捷启动方式真的很不方便，所以去网上找了一个之前的带有此功能的版本，然后就想办法添加JumpList到新版本中，首先得找到运行这几种模式的命令行参数，通过使用一下命令可以找出运行参数

[shell gutter="false"]
wmic process where caption=&quot;winsnap.exe&quot; get caption,commandline /value
[/shell]

五种模式的运行参数如下

[shell gutter="false"]
&quot;D:\Program_Files\WinSnapPortable\App\WinSnap\WinSnap.exe&quot; /tbswitch /fullscreen
&quot;D:\Program_Files\WinSnapPortable\App\WinSnap\WinSnap.exe&quot; /tbswitch /application
&quot;D:\Program_Files\WinSnapPortable\App\WinSnap\WinSnap.exe&quot; /tbswitch /window
&quot;D:\Program_Files\WinSnapPortable\App\WinSnap\WinSnap.exe&quot; /tbswitch /object
&quot;D:\Program_Files\WinSnapPortable\App\WinSnap\WinSnap.exe&quot; /tbswitch /region
[/shell]

接下来就需要找办法添加/修改JumpList
其中一些很好的软件有
1. [JumplistExtender](https://code.google.com/p/jumplist-extender/)(开源的，有空可以学学JumpList)，可以在任意程序上添加任意JumpList
[![jumplist-extender](http://202.203.209.55:8080/wp-content/uploads/2014/10/jumplist-extender-300x226.jpg)](http://202.203.209.55:8080/wp-content/uploads/2014/10/jumplist-extender.jpg)
2. [JumpListEditor](http://jumplisteditor.blogspot.com/)，与JumplistExtender功能差不多
[![JumpListEditor](http://202.203.209.55:8080/wp-content/uploads/2014/10/JumpListEditor-300x119.png)](http://202.203.209.55:8080/wp-content/uploads/2014/10/JumpListEditor.png)
3\. [jumplist-launcher](http://en.www.ali.dj/jumplist-launcher/)，可以在此程序上方便的添加任意JumpList
[![jumplistlauncher](http://202.203.209.55:8080/wp-content/uploads/2014/10/jumplistlauncher-228x300.png)](http://202.203.209.55:8080/wp-content/uploads/2014/10/jumplistlauncher.png)
4\. [Jump List Manager](http://www.addictivetips.com/windows-tips/manage-windows-7-jump-list-items-in-taskbar-with-jump-list-manager/)，与jumplist-launcher功能差不多
[![jump-list-manager](http://202.203.209.55:8080/wp-content/uploads/2014/10/jump-list-manager-300x234.jpg)](http://202.203.209.55:8080/wp-content/uploads/2014/10/jump-list-manager.jpg)

**不过目前暂时没找到一款可以修改已存在JumpList的程序，使用以上1、2中的软件添加新的JumpList会覆盖已有的，自己简略的看过一点点这方面的资料，感觉这水还是有点小深，后面如果有时间，想在[JumplistExtender](https://code.google.com/p/jumplist-extender/)上完善一下添加可以修改已有的JumpList得功能，那就完美了！**

如果想深入学习JumpList可参考
1\. [jump-lists](http://windows.microsoft.com/en-us/windows7/products/features/jump-lists)
2\. [jump_lists_view](http://www.nirsoft.net/utils/jump_lists_view.html)

&nbsp;