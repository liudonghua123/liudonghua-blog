---
title: wordpress 修改默认字体
tags:
  - css
  - font-family
  - wordpress
id: 161
categories:
  - 学习
date: 2014-10-15 15:14:55
---

wordpress中虽然有很多修改字体的插件，但都不能很好的支持中文字体，有些是上传自己的字体，但这种方式随便一个中文字体文件都有10M左右，严重影响网页加载；有些是使用Google Fonts，但都没有很好的中文字体支持，所以最好的方式是修改网页里font-family使用系统自带的字体。
中文版系统中网页里面默认使用的字体都不好看，如Windows中默认使用宋体就非常难看，所以这篇文章分享一下我自己修改字体的方法，主要分为修改前台博客里面的字体以及修改TinyMCE中文本编辑框中的字体。

<!--more-->

1. 修改前台博客里面的字体
直接修改主题的style.css文件，替换里面的font-family为

[css]
font-family: Avenir, 'Lucida Grande', Calibri, Helvetica, Arial, 'Microsoft YaHei', 'Hiragino Sans GB', SimHei, STHeiti, sans-serif;
[/css]

这里需要提到一点是font-family中其实不需要为了考虑中文版的系统使用中文版的字体名称，如下所示

[css]
font-family: Avenir, 'Lucida Grande', Calibri, Helvetica, Arial, 'Microsoft YaHei', '微软雅黑', 'Hiragino Sans GB', '冬青黑体', SimHei, STHeiti, sans-serif;
[/css]

2. 修改TinyMCE中文本编辑框中的字体
这需要根据自己使用的主题，如我使用的eighties主题，没找到后台管理页面的修改地方，只能直接修改相应的css文件，我修改的是wp-content/themes/eighties/css/editor.css
同样修改里面的font-family
这样写文章时显示的字体也漂亮很多了，如下图所示
[![wordpress_tinymce_font-family](/resources/2014/10/wordpress_tinymce_font-family.png)](/resources/2014/10/wordpress_tinymce_font-family.png)