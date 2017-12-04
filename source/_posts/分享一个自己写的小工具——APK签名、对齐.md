---
title: 分享一个自己写的小工具——APK签名、对齐
tags:
  - javafx
  - signapk
id: 672
categories:
  - 学习
date: 2015-03-10 20:28:35
---

似乎已经有好久没有写博客了，这个习惯可不好![](http://202.203.209.55:8080/wp-content/plugins/wp-emoji-one/icons/1F600.png)
这次分享一个自己写的小工具，对apk（其实zip格式的文件也是支持的，如Chrome的扩展crx其实也是一个zip文件，并且发布时也是要签名的）进行签名、对齐操作。

<!--more-->

支持的特性：
1. 可以动态切换应用主题，默认有白色和黑色主题
2. 可以动态添加签名使用的key，默认有media、platform、shared、testkey这四种
3. 可以动态设置应用语言，目前支持中文和英文
4. 支持拖拽的方式选择输入文件或移动输出文件
以上如果需要修改或添加可以参考resources目录中theme、certs、locales子目录里的内容，非常简单，添加修改后重启即可生效！

欢迎大家改进，代码在[这里](https://github.com/liudonghua123/signapk_fx "https://github.com/liudonghua123/signapk_fx")！

实现是基于JavaFx，其中的对话框controlsfx原来是不在SDK中的，现在已经在JDK 8 update 40中已经包括了（可参考[JavaFX Dialogs](http://code.makery.ch/blog/javafx-dialogs-official/)）！
所以需要安装JDK8u40或以上版本
然后在Eclipse中通过Maven工程导入即可。

附一张截图
[![signapk_main_window_preview](/resources/2015/03/main_window_preview.png)](/resources/2015/03/main_window_preview.png)