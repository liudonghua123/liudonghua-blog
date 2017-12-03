---
title: 自己制作Eclipse插件安装包
tags:
  - Java Web
id: 9
categories:
  - 学习
date: 2014-10-02 21:56:14
---

之前在eclipse中使用[run-jetty-run](https://code.google.com/p/run-jetty-run/ "run-jetty-run")还是挺不错的，相比tomcat要轻量级一些，但最近更新eclipse了想要重装这个插件（[update site](http://run-jetty-run.googlecode.com/svn/trunk/updatesite)），但不管设不设代理总是无法安装，最后看了一下update site网页里的内容似乎是包括了插件的内容，于是就想着自己制作一个离线安装包吧！
需要用到的文件及下载网址如下

<!--more-->

[site.xml
](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/site.xml)[content.jar
](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/content.jar)[artifacts.jar
](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/artifacts.jar)[runjettyrun_1.3.3.201203161919.jar
](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/plugins/runjettyrun_1.3.3.201203161919.jar)[runjettyrun.jetty8_1.3.3.201203161919.jar
](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/plugins/runjettyrun.jetty8_1.3.3.201203161919.jar)[runjettyrun.jetty7_1.3.3.201203161919.jar](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/plugins/runjettyrun.jetty7_1.3.3.201203161919.jar)[
](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/features/runjettyrun_feature_support_jetty7_1.3.3.201203161919.jar)[runjettyrun_feature_1.3.3.201203161919.jar](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/features/runjettyrun_feature_1.3.3.201203161919.jar)[
](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/plugins/runjettyrun.jetty7_1.3.3.201203161919.jar)[runjettyrun_feature_support_jetty8_1.3.3.201203161919.jar
](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/features/runjettyrun_feature_support_jetty8_1.3.3.201203161919.jar)[runjettyrun_feature_support_jetty7_1.3.3.201203161919.jar](https://run-jetty-run.googlecode.com/svn/trunk/updatesite/features/runjettyrun_feature_support_jetty7_1.3.3.201203161919.jar)

最后再把这几个文件打包成一个zip文件
<span style="line-height: 1.5;">目录层次结构如下</span>

xxx.zip(附上我自己做的[runjettyrun_1.3.3.201203161919](/resources/2014/10/runjettyrun_1.3.3.201203161919.zip)方便懒人下载 )
site.xml
content.jar
artifacts.jar
plugins/runjettyrun_1.3.3.201203161919.jar
plugins/runjettyrun.jetty8_1.3.3.201203161919.jar
plugins/runjettyrun.jetty7_1.3.3.201203161919.jar
features/runjettyrun_feature_support_jetty8_1.3.3.201203161919.jar
features/runjettyrun_feature_support_jetty7_1.3.3.201203161919.jar
features/runjettyrun_feature_1.3.3.201203161919.jar

最后就只需要断网（最好断网，不然还会联网去尝试下载更新）然后选择本地文件安装即可。