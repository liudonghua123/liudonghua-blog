---
title: Windows和Linux通过主机名互相访问
tags:
  - libnss-winbind
  - netbios
  - winbind
id: 636
categories:
  - 学习
date: 2014-12-08 12:20:43
---

通常在一个局域网里都是安装有DHCP服务器，各主机的IP都是动态获取的，如果想访问某一台主机的服务（如共享文件、HTTP、FTP等），使用IP是一件很痛苦的事，并且IP也是会变化的。

<!--more-->当然Windows主机之间默认可以通过主机名（在cmd中通过hostname查看）访问（使用[NetBIOS](http://en.wikipedia.org/wiki/NetBIOS)协议），但是默认情况下Windows和Linux之间不能通过主机名互相访问，所以这里简单讲解一下怎样解决Windows-Linux通过主机名互相访问的问题。

1\. 在Linux（Ubuntu）下访问Windows
安装winbind（libnss-winbind），修改/etc/nsswitch.conf，在hosts段后面添加wins
我修改之后的内容为: "hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4 wins"
网上很多教程到这一步就完成了，但也有很多人到这里还是不能通过主机名ping通Windows机器，这时一般再安装一个libnss-winbind就可以解决问题

2\. Windows下访问Linux
还是只需在Linux下操作，安装一个samba即可，不过我遇到的问题是，在Windows ping ubuntu主机名时如果ubuntu主机名结尾没有".local"，ping时需要带上".local"，有的话直接ping 带".local"主机名。

参考资料
1\. [HowTo: Configure Ubuntu to be able to use and respond to NetBIOS hostname queries like Windows does](http://www.serenux.com/2009/09/howto-configure-ubuntu-to-be-able-to-use-and-respond-to-netbios-hostname-queries-like-windows-does/)
2\. [一个局域网内，没有dns服务器时，机器名到ip的解析由谁来负责呢？](http://bbs.51cto.com/viewthread.php?tid=730638&page=1#pid3633721)
3. [Windows hostnames are not resolved](http://askubuntu.com/questions/93302/windows-hostnames-are-not-resolved/380521#380521)
4\. [ubuntu-server-not-resolving-lan-hostnames](http://askubuntu.com/questions/244865/ubuntu-server-not-resolving-lan-hostnames)