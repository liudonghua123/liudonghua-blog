---
title: Windows下创建快捷方式
tags:
  - mklink
  - Windows
id: 187
categories:
  - 学习
date: 2014-10-17 11:05:00
---

在Windows下创建快捷方式，可以右键-新建-快捷方式，然后指向目的位置，或者右键拖拽文件或文件夹到目的位置，释放右键选择创建快捷方式，但这种方式创建的快捷方式是有后缀的（lnk后缀），并且不能修改（move修改后就变成一个打不开的文件）。<!--more-->
那Windows有没有与Linux/Unix类似的命令行工具可以创建不带后缀的快捷方式吗？
答案就是mklink

[shell]
mklink

Creates a symbolic link.

MKLINK [[/D] | [/H] | [/J]] Link Target
/D Creates a directory symbolic link. Default is a file symbolic link.
/H Creates a hard link instead of a symbolic link.
/J Creates a Directory Junction.
Link specifies the new symbolic link name.
Target specifies the path (relative or absolute) that the new link refers to.
[/shell]

/D、/H、/J选项只能用一个
/H 创建硬链接，只能针对文件，如果是文件夹，则会提示"Access is denied."
/D、/J 只能针对文件夹，如果是文件则创建之后快捷方式是文件夹图标，点击提示"xxx is not accessable. The directory is invalid"

下面就说一下/D /J的区别
/D /J在本地时几乎没区别
如我创建了两个访问相同目标的链接，访问时没有任何区别

[shell]
D:\android&gt;dir
Volume in drive D is Program
Volume Serial Number is 04D9-9235
Directory of D:\android
2014-10-17 10:51 &lt;DIR&gt; .
2014-10-17 10:51 &lt;DIR&gt; ..
2014-09-11 21:11 &lt;DIR&gt; android-ndk-r10b
2014-10-17 10:28 &lt;JUNCTION&gt; android-sdk-directory-Junction [D:\android\android-studio\sdk]
2014-10-17 10:27 &lt;SYMLINKD&gt; android-sdk-directory-symbolic-link [D:\android\android-studio\sdk]
2014-10-17 10:26 &lt;DIR&gt; android-studio
0 File(s) 0 bytes
6 Dir(s) 125,838,143,488 bytes free
[/shell]

但这两者确实存在着一些区别，详见[directory-junction-vs-directory-symbolic-link](http://superuser.com/questions/343074/directory-junction-vs-directory-symbolic-link)
1\. /D需要管理员权限，/J普通权限即可
2\. 如果访问远程电脑上的Directory Symbolic Link，其解析实在远程电脑上，而如果是Directory Junction Link，则其解析是本地，例如在Alice电脑上
C:/juncationlink -&gt; C:/onesharedirectory
C:/symboliclink  -&gt; C:/onesharedirectory
这时如果在Bob电脑上访问这两个目录
\\Alice\c$\juncationlink -&gt; \\Alice\c$\onesharedirectory
\\Alice\c$\symboliclink  -&gt; \\Bob\c$\onesharedirectory
显然使用symboliclink是访问错误的

/D /J 两者都还有一个共同点，可以未卜先知，及可以创建一个当时不存在的链接，而/H就不行