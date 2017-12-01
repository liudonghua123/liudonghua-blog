---
title: 网络安装操作系统简述
tags:
  - OS
id: 534
categories:
  - 学习
date: 2014-11-14 15:16:01
---

通常我们都是通过光盘、移动硬盘或U盘来安装操作系统的，但其实还有一种对普通用户来说不常见的安装方式，那就是通过网络来安装。<!--more-->

网络安装其实也是有很多方式，这里只提供一种方法的概述，详见链接指向的文档，其他方法（如使用微软的MicrosoftDeploymentToolki，功能比较强大，但配置也非常繁琐）后面有机会再补上。

在一个局域网里，一般都已经有一台DHCP服务器，而普通网络安装要求把DHCP服务器也作为TFTP服务器使用，这种方式比较简单（也可以直接把两台电脑通过一根网线连接，然后一台电脑上同时启动DHCP、TFTP给另一台待安装系统的电脑提供服务，可参考"[how-to-boot-from-the-network-pxe-boot-with-tftp-and-windows-pe](http://blog.ryantadams.com/2008/02/01/how-to-boot-from-the-network-pxe-boot-with-tftp-and-windows-pe/)"）。但如果局域网已有DHCP服务器或没有办法那台DHCP服务器，就只能使用[proxyDHCP](http://en.wikipedia.org/wiki/Preboot_Execution_Environment)，这里使用一个软件[Serva](http://www.vercot.com/~serva/default.html)（功能很强大，可以作为HTTP/FTP/TFTP/DHCP/proxyDHCP/...等服务器使用），参考"[How to Install Any Version of Windows from Other Network Computers](http://www.7tutorials.com/how-install-any-version-windows-other-network-computers)"。这种方法我已试验过，是可行的，但要注意如果使用免费版，会有一些连接限制，局域网里电脑（并不仅仅是待安装系统的电脑）太多，可以先网络模式启动待安装系统的电脑，然后再运行Serva。

DHCP v.s. proxyDHCP
[![Windows1_DHCP_m](http://202.203.209.55:8080/wp-content/uploads/2014/11/Windows1_DHCP_m.png)](http://202.203.209.55:8080/wp-content/uploads/2014/11/Windows1_DHCP_m.png "http://www.vercot.com/~serva/an/WindowsPXE1.html")