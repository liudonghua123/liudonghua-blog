---
title: 正则表达式备忘
tags:
  - regex
id: 15
categories:
  - 学习
date: 2014-10-03 14:58:16
---

正则表达式主要分为POSIX和Perl风格两种
POSIX包括[POSIX Basic Regular Expressions](http://www.regular-expressions.info/posix.html)和其后扩展的[POSIX Extended Regular Expressions
](http://en.wikibooks.org/wiki/Regular_Expressions/POSIX_Extended_Regular_Expressions)<!--more-->

Unix/Linux命令中通常使用POSIX风格的，有基本的和扩展的。

Perl风格的包括纯[Perl](https://www.cs.tut.fi/~jkorpela/perl/regexp.html)风格和[Perl Compatible Regular Expressions
](http://en.wikibooks.org/wiki/Regular_Expressions/Perl_Compatible_Regular_Expressions)

其他一些语言如[Java](http://www.journaldev.com/634/java-regular-expression-tutorial-with-examples), [JavaScript](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions), Ruby, Python, PHP, Microsoft's .NET Framework都是使用类似于Perl风格的，即[Perl Compatible Regular Expressions](http://en.wikibooks.org/wiki/Regular_Expressions/Perl_Compatible_Regular_Expressions)。

学习、测试正则表达式的一个非常好网站[regexr](http://regexr.com/)

最近学习到一个容易混淆的知识点——greedy vs. non-greedy(ungreedy)

?通常情况用于匹配0次或1次，但如果置于*, +, ?, {}这些后面则表示non-greedy
如下面例子所示
如我需要在一行文本里(&lt;img src=\"image.url\"&gt;&lt;a href="anchor.url"&gt;anchor&lt;/a&gt;)仅仅匹配&lt;img&gt;标签
使用[&lt;img\s.*&gt;](http://regexr.com/39k4t)则会同时匹配后面的&lt;a&gt;
[![greedy_regex](http://202.203.209.55:8080/wp-content/uploads/2014/10/greedy_regex.png)](http://202.203.209.55:8080/wp-content/uploads/2014/10/greedy_regex.png)
使用[&lt;img\s.*?&gt;](http://regexr.com/39k4q)则可避免这个问题
[![non-greedy_regex](http://202.203.209.55:8080/wp-content/uploads/2014/10/non-greedy_regex.png)](http://202.203.209.55:8080/wp-content/uploads/2014/10/non-greedy_regex.png)