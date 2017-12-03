---
title: 使用代理下载或更新Android源码
tags:
  - android
  - proxy
  - squid
id: 603
categories:
  - 学习
date: 2014-12-03 10:16:27
---

之前在AWS的EC2上下载Android源码，然后再下载到本地很麻烦，并且本地源码更新也经常会出现网络连接问题，后来突然想到不妨在AWS上设置代理，然后连接AWS上的代理更新本地代码。<!--more-->

所以今天就尝试一下，由于我的AWS上EC2安装的是ubuntu系统，所以搜索了一下ubuntu下的代理服务器，一般都是用squid，那就这个吧。以下我以最少/最简配置说明，毕竟只是临时的。执行以下命令安装
```shell
sudo apt-get update
sudo apt-get install squid3
sudo cp /etc/squid3/squid.conf /etc/squid3/squid.conf.original
sudo chmod a-w /etc/squid3/squid.conf.original
```
然后修改/etc/squid3/squid.conf配置文件
端口默认是http_port 3128
acl默认配置了一些允许或禁止访问规则（详见配置文件中的http_access allow xxx/http_access deny xxx），我为了简单，把http_access deny all注释掉，然后最后加上http_access allow all。然后重启squid服务器sudo service squid3 restart，这样远程代理服务器就配置好了。

本地只要在执行repo init或repo sync之前设置两个环境变量即可
export http_proxy=http://your_aws_public_up:your_port
export https_proxy=http://your_aws_public_up:your_port
这样就可以无各种抓狂的网络问题的去下载/同步Android源码了，赶紧试一下去吧！

其实squid功能很强大，acl设置可以限定ip/ip段/时间/...，还可以加上用户验证（用户名+密码）等。详见以下一些网址内容！

参考资料
1\. [Squid - Proxy Server](https://help.ubuntu.com/14.04/serverguide/squid.html)
2\. [Creating an HTTP Proxy Using Squid on Ubuntu 12.04](https://www.linode.com/docs/networking/squid/squid-http-proxy-ubuntu-12-04)
3\. [Squid](http://wiki.ubuntu.com.cn/Squid)