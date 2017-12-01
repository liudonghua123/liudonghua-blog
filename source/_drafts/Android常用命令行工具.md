---
title: Android常用命令行工具
id: 734
categories:
  - 学习
tags:
---

ANDROID_ID.

This is how you can query its value via adb shell:
settings get secure android_id
Or by querying the settings content provider:
content query --uri content://settings/secure/android_id --projection value

参考资料
1\. [Getting Android Device Identifier From ADB and Android SDK](http://stackoverflow.com/questions/5486694/getting-android-device-identifier-from-adb-and-android-sdk)