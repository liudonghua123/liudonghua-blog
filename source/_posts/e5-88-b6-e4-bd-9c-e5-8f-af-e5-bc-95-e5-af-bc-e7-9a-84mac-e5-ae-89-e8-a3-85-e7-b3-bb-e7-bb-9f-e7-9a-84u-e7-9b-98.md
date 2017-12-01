---
title: 制作可引导的Mac安装系统的U盘
tags:
  - installer
  - mac
  - system
id: 153
categories:
  - 学习
date: 2014-10-15 14:34:47
---

这方面的文章网上其实已经有很多了，但多数属于以下方法一，有些麻烦，这里就不再重述。<!--more-->
这里只是总结并再提供一种更方便的方法——使用一条命令或者使用[DiskMaker X](http://diskmakerx.com)轻松点几下。

方法一. 完整的常用方法详解
[How to make a bootable Mac OS X 10.10 Yosemite install drive using a USB Stick](http://www.macworld.co.uk/how-to/mac/make-bootable-mac-os-x-1010-yosemite-install-drive-3575875/)

方法二. 使用一条命令方法（适用于>=10.7），来自于[这里](https://kb.iu.edu/d/bbdj "How do I create install media for Mac OS X 10.7 or later?")以及[这里](http://osxdaily.com/2014/10/16/make-os-x-yosemite-boot-install-drive/)，实际使用时要按照自己的系统环境

```shell
sudo /Applications/Install\ OS\ X\ Yosemite.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled --applicationpath /Applications/Install\ OS\ X\ Yosemite.app --nointeraction
```

方法三. 使用[DiskMaker X](http://diskmakerx.com)