---
title: Windows的apt-get或yum或zypper或brew——chocolatey
tags:
  - chocolatey
  - Windows
id: 578
categories:
  - 学习
date: 2014-11-29 15:00:05
---

使用Linux、类Unix的等系统的方便之处不仅在于各种方便实用的小命令，而且搜索、安装（自动解决依赖关系）、更新、卸载软件的快捷——这就是软件包管理器。<!--more-->

如Ubuntu/Debian使用apt-get来管理deb格式的安装包，Red Hat/Fedora/CentOS使用yum来管理[rpm](http://en.wikipedia.org/wiki/RPM_Package_Manager)格式的安装包，SUSE使用[Zypper](http://doc.opensuse.org/documentation/html/openSUSE_114/opensuse-reference/cha.sw_cl.html)来管理rpm格式的安装包；Mac则使用brew自动下载源码、编译、安装，并且也提供搜索、更新、卸载等功能。

Windows安装软件的方式一般要下载安装包（exe、msi等格式）或直接绿色软件解压即运行，安装、搜索、更新、卸载等都非常不方便，今天在找Windows版的curl时发现一个类似以上软件包管理器的Windows版本——[chocolatey](https://chocolatey.org/)，听名字就非常“可口”，虽然可用的安装源远没有以上的那些丰富，而且还有如安装cinst vim时会弹出图形界面来让点确定才能继续（安装包的静默安装方式做的不好）。

安装chocolatey的方式，以管理员权限运行cmd，然后在其中执行

[shell]
@powershell -NoProfile -ExecutionPolicy unrestricted -Command &quot;iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))&quot; &amp;&amp; SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
[/shell]

其实就是调用powershell，也可以直接在powershell中执行

[shell]
iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))
[/shell]

安装好chocolatey之后以后安装什么软件就可以直接运行"choco install **package_name**"，如安装git直接调用"choco install git"，安装vlc直接"choco install vlc"，安装强大的[spf13-vim](https://github.com/liudonghua123/spf13-vim)(vim+bundles)要使用以下多条命令(chocolatey还不支持自动解决依赖关系)

[shell]
cinst git
cinst curl
cinst ctags
cinst spf13.vim
[/shell]

这里附上Linux常用包管理的运行参数
[![linux-package-management-cheatsheet](http://202.203.209.55:8080/wp-content/uploads/2014/11/linux-package-management-cheatsheet.png)](http://202.203.209.55:8080/wp-content/uploads/2014/11/linux-package-management-cheatsheet.png)

参考资料：
1. [chocolatey](https://chocolatey.org/)
2. [linux-package-management-cheatsheet](http://danilodellaquila.com/blog/linux-package-management-cheatsheet)
3. [how-to-use-vundle-to-manage-vim-plugins-on-a-linux-vps](https://www.digitalocean.com/community/tutorials/how-to-use-vundle-to-manage-vim-plugins-on-a-linux-vps)