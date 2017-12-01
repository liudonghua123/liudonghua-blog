---
title: Java中的REPL
id: 772
categories:
  - 学习
date: 2015-05-10 15:03:27
tags:
---

Ruby、Python、PHP、JavaScript等语言都有一种interactive shell的概念，即可以在命令行中执行代码，并且立刻看到效果，这种特性也叫做[REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)（Read–eval–print_loop）。通过这种方式可以很快的学习或验证一些语言特性，例如执行arr=new ArrayList&lt;String&gt;();然后就可以看到arr: []，再输入arr.按两下Tab就可以列举出arr这个ArrayList对象可支持的方法。
<!--more-->

这里有一个[Online-REPs-and-REPLs](http://joel.franusic.com/Online-REPs-and-REPLs/)汇总的列表

对于Java领域，目前还没有类似于Ruby、Python等成熟的REPL，只有一些接近的

1. [beanshell](http://www.beanshell.org/)
支持命令行（java -cp ".;bsh-2.0b4.jar" bsh.Interpreter）以及图形化界面（java -jar bsh-2.0b4.jar 或java -cp ".;bsh-2.0b4.jar" bsh.Console），其源码在[googlecode](https://code.google.com/a/apache-extras.org/p/beanshell/)上或[github](https://github.com/pejobo/beanshell2)上
[BeanShell, Simple Java Scripting](http://www.beanshell.org/manual/bshmanual.html)
[BeanShellSlides
](http://www.beanshell.org/BeanShellSlides.pdf)如下是一个简单的运行实例

[shell]
E:\Downloads&gt;java -cp &quot;.;bsh-2.0b4.jar&quot; bsh.Interpreter
BeanShell 2.0b4 - by Pat Niemeyer (pat@pat.net)
bsh % arr=new ArrayList();
bsh % arr.add(1);
bsh % arr.add(2);
bsh % arr.add(3);
bsh % arr;
bsh % print arr;
// Error: EvalError: Typed variable declaration : Class: print not found in namespace : at Line: 6 :
 in file: &lt;unknown file&gt; : print

bsh % print(arr);
[1, 2, 3]
bsh %
[/shell]

2. [javarepl](http://www.javarepl.com/console.html)
仅支持命令行，可以从其[Github](https://github.com/albertlatacz/java-repl)上下载
如下是一个简单的运行实例

[shell]
E:\Downloads&gt;java -jar javarepl.jar
Welcome to JavaREPL version 278 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_40)
Type expression to evaluate, :help for more options or press tab to auto-complete.
java&gt; :help
Available commands:
    :help - shows this help
    :cls - clear screen
    :quit - quit application
    :hist [num] - shows the history (optional 'num' is number of evaluations to show)
    :h? &lt;term&gt; - searches the history
    :h! num - evaluate expression from history
    :reset - resets environment to initial state
    :replay - replay all evaluations
    :eval &lt;path&gt; - evaluates all expressions from file (expression per line)
    :cp &lt;path&gt; - includes given file or directory in the classpath
    :load &lt;path&gt; - loads source file
    :list &lt;results|types|methods|imports|all&gt; - list specified values
    :src - show source of last evaluated expression
    :type &lt;expression&gt; - shows the type of an expression without affecting current context
    :check &lt;syntax&gt; - checks syntax of given single line expression and returns detailed report

java&gt; arr=new ArrayList&lt;String&gt;();
java.util.ArrayList arr = []
java&gt; arr.add(&quot;String&quot;);
java.lang.Boolean res1 = true
java&gt; arr.add(someUndefinedNum);
ERROR: cannot find symbol
  symbol:   variable someUndefinedNum
  location: class Evaluation
    arr.add(someUndefinedNum);;
            ^
java&gt; arr.add(123);
java.lang.Boolean res2 = true
java&gt; print (arr);
ERROR: cannot find symbol
  symbol:   method print(java.util.ArrayList)
  location: class Evaluation
    print (arr);;
    ^
java&gt; arr;
java.util.ArrayList res3 = [String, 123]
java&gt;
[/shell]

当然还有一些其他的，如[spring-shell](http://docs.spring.io/spring-shell/docs/current/reference/htmlsingle/)、[drjava](http://drjava.org/)、[groovyconsole](http://www.groovy-lang.org/groovyconsole.html)、[scala](http://www.scala-lang.org/)等等的。