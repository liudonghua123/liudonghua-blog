---
title: Android开发中的一个奇怪的问题
tags:
  - android
  - android studio
  - gradle
  - java
id: 175
categories:
  - 学习
date: 2014-10-16 21:12:36
---

今天在写一个小Demo时，遇到一个很奇怪的问题，弄了很久最终才勉强解决，问题是这样的，在子类中调用父类的一个属性的方法始终不行，但在父类里面可以调用

<!--more-->

如下图所示
父类里方法实现
[![strange_base](/resources/2014/10/strange_base.png)
](/resources/2014/10/strange_base.png)子类里重写这个方法
[![strange_problem](/resources/2014/10/strange_problem.png)
](/resources/2014/10/strange_problem.png)代码提示中是可以看到有这个方法的
[![strange_problem_2](/resources/2014/10/strange_problem_2.png)
](/resources/2014/10/strange_problem_2.png)但编译时报错

[text]
incompatible types: LayoutParams cannot be converted to int
mTabLayout.addView(tabView, new LinearLayout.LayoutParams(0, MATCH_PARENT, 1));
[/text]

为了获取更详细错误信息，在命令行上执行

```shell
gradle assembleDebug
```

但提示如下图所示错误
[![problem_1](/resources/2014/10/problem_1.png)](/resources/2014/10/problem_1.png)
即使修改成

```java
mTabLayout.addView(tabView);
```

也会提示如下错误
[![problem_2](/resources/2014/10/problem_2.png)](/resources/2014/10/problem_2.png)

用反射也不行，直接到这里就报错了

```java
mTabLayout.getClass().getMethod...
```

最后终于用这种方式可以勉强通过，即**强制类型转换一下**
[![strange_solve](/resources/2014/10/strange_solve.png)](/resources/2014/10/strange_solve.png)

其实我在运行时可以执行一下这句，并且没有错误

```java
mTabLayout.addView(tabView, new LinearLayout.LayoutParams(0, MATCH_PARENT, 1));
```