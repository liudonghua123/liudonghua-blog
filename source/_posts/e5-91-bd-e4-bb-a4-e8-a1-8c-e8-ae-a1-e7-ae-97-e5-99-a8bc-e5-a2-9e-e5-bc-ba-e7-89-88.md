---
title: 命令行计算器bc增强版
tags:
  - bc
id: 391
categories:
  - 学习
date: 2014-10-29 10:30:25
---

bc是一个非常好用并且功能强大的命令行计算器，有悠久的历史，但使用时或多或少遇到一些"问题"，所以就想修改源码，增加或修改一些特性，以便更好的方便我们在命令行上使用。
<!--more-->目前遇到的一些不方便如下所示

1\. 转换一个大于10进制的数时，英文字母必须大写

[shell gutter="false"]
liudonghua@www:~$ echo 'obase=14;ibase=15;2e' | bc
(standard_in) 1: syntax error
liudonghua@www:~$ echo 'obase=14;ibase=15;2E' | bc
32
[/shell]

2\. 把一个数转换成大于16进制的数时，则不会使用字母代替

[shell gutter="false"]
liudonghua@www:~$ echo 'obase=14;ibase=15;28' | bc
2A
liudonghua@www:~$ echo 'obase=16;ibase=15;28' | bc
2C
liudonghua@www:~$ echo 'obase=17;ibase=15;2E' | bc
 02 10
[/shell]

3\. 例如把2进制数10转换成10进制，用以下第一种方式不行，可以使用第二种或第三种

[shell gutter="false"]
liudonghua@www:~$ echo 'ibase=2;obase=10;10' | bc
10
liudonghua@www:~$ echo 'ibase=2;obase=A;10' | bc
2
liudonghua@www:~$ echo 'obase=10;ibase=2;10' | bc
2
[/shell]

所以目前暂时计划添加这三项特性（目前才开始，只能抽空实现）
1\. 进制数表示的字母不区分大小写
2\. 支持大于16进制的数表示为数字+字母表示形式（通过添加参数/标志实现）
3\. 默认取消obase里的值受ibase影响（通过添加参数/标志实现）

后面如果有新的需求再更新，项目详见[enhanced-bc](https://github.com/liudonghua123/enhanced-bc)。