---
title: 'Android开发备忘[持续更新]'
tags:
  - android
  - Chrome
  - Chrome Extension
id: 23
categories:
  - 学习
date: 2014-10-08 09:51:28
---

1\. 直接在网页中下载Google Play Store中的APK，只需要知道APK的package name（即访问访问play的url的参数id的值，如https://play.google.com/store/apps/details?id=me.bpear.archonpackager的package name即me.bpear.archonpackager）
http://apps.evozi.com/apk-downloader/<!--more-->

2\. 在Chrome OS/Chrome中运行android程序
详见
https://github.com/vladikoff/chromeos-apk
https://github.com/vladikoff/chromeos-apk/blob/master/archon.md

简言之就是就是安装Archon运行环境（Chrome OS已自带），然后安装封装好的APK的Chrome扩展crx（由于Chrome的官方市场尚未提供这类应用，只能在扩展程序-开发者模式载入正在开发的应用方式安装）

封装APK为Chrome扩展可以有以下三种方式（目前[支持的应用列表](https://docs.google.com/spreadsheets/d/1iIbxaftAu_ho5rv9fUlXSLTzwU6MbKOldsWXyrYiyo8/htmlview?usp=sharing&sle=true)）
1\. 在手机中安装[ARChon Packager
](https://play.google.com/store/apps/details?id=me.bpear.archonpackager)

2\. 在电脑上安装chromeos-apk（需要nodejs支持）
sudo npm install chromeos-apk -g

3\. [手动封装](http://lifehacker.com/how-to-run-android-apps-inside-chrome-on-any-desktop-op-1637564101)（详见Alternative: Convert APKs Manually）