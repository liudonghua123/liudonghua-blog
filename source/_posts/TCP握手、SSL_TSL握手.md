---
title: TCP握手、SSL/TSL握手
tags:
  - handshake
  - SSL/TSL
  - TCP
id: 115
categories:
  - 学习
date: 2014-10-14 21:39:47
---

[toc]
这里简单介绍一些TCP/SSL/TSL中的握手机制

<!--more-->

# TCP建立连接三次握手

由于TCP是全双工的，在TCP中通信中两端是对等的，即不区分客户端/服务端，从一端（如A）连通到另一端（如B），需要A发送一个建立连接信息（SYNchronize sequence number），B收到后返回一个确认收到信息(ACKnowledgement of SYN)，此时A收到确认收到信息则可确信从A->B是连通的，类似还需要确信B->A也是连通的，所以理论上建立连接需要四次握手，但其实为了提高效率B收到A建立连接信息（建立A->B连接）时可以返回A的确认收到信息以及B的建立连接信息（建立B->A连接）,最后A收到B返回的信息后再返回给B一个确认收到信息，最后B收到后，此时双方就正式建立连接。
如下图所示
[![tcp3waysynch](/resources/2014/10/tcp3waysynch.png)](/resources/2014/10/tcp3waysynch.png)

# TCP断开连接四次握手

这与建立连接的三次握手类似，其实在一些特殊情况（如下图Server接收到#1 FIN时服务端已没有需要发送给Client端的数据，此时#2 ACK和下面的#1 FIN就会合并）下也会变成三次握手，详见[Transmission_Control_Protocol](http://en.wikipedia.org/wiki/Transmission_Control_Protocol)
[![tcpclose](/resources/2014/10/tcpclose.png)](/resources/2014/10/tcpclose.png)

# SSL/TSL握手

SSL/TSL握手相对来说就复杂一些，这里就转载几篇写的不错的文章
[SSL Handshake Steps In Detail
](http://www.pierobon.org/ssl/ch2/detail.htm)[An Introduction to Mutual SSL Authentication](http://www.codeproject.com/Articles/326574/An-Introduction-to-Mutual-SSL-Authentication)

![mutualssl_small](/resources/2014/10/mutualssl_small.png)

## 不需要客户端认证的SSL握手[
](http://www.codeproject.com/Articles/326574/An-Introduction-to-Mutual-SSL-Authentication)[![1WaySSL](/resources/2014/10/1WaySSL.png)
](/resources/2014/10/1WaySSL.png)

## 需要客户端认证的SSL握手

[![2WaySSL](/resources/2014/10/2WaySSL.png)](/resources/2014/10/2WaySSL.png)[
](http://www.codeproject.com/Articles/326574/An-Introduction-to-Mutual-SSL-Authentication)