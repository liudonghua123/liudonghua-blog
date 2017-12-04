---
title: Javac编译中的调试信息
tags:
  - debug
  - java
  - javac
id: 501
categories:
  - 学习
date: 2014-11-09 19:25:37
---

有时候debug单步跟踪时，特别是进入到系统库的实现中想查看某个变量的值，这时候就会出现"xxx cannot be resolved to a variable"，对于调试来说很郁闷。如下图所示

[![java_debug_vars](/resources/2014/11/java_debug_vars.png)](/resources/2014/11/java_debug_vars.png)
但是this是可以查看具体调试信息
[![java_debug_this](/resources/2014/11/java_debug_this.png)](/resources/2014/11/java_debug_this.png)

<!--more-->

所以就深入看了一下为什么会出现这个问题
以下是javac编译时的选项
[![javac_help](/resources/2014/11/javac_help.png)](/resources/2014/11/javac_help.png)
下面以一个简单的小例子编译后查看class文件结构信息
1. 默认选项，即不加任何选项
[![javac_sample_default](/resources/2014/11/javac_sample_default.png)](/resources/2014/11/javac_sample_default.png)
2. 使用-g选项，即包括所有调试信息
[![javac_sample_g](/resources/2014/11/javac_sample_g.png)](/resources/2014/11/javac_sample_g.png)
3. 使用-g:none选项，即不包括任何调试信息
[![javac_sample_gnone](/resources/2014/11/javac_sample_gnone.png)](/resources/2014/11/javac_sample_gnone.png)
4. 使用-g:lines选项，即只包括lines调试信息
[![javac_sample_glines](/resources/2014/11/javac_sample_glines.png)](/resources/2014/11/javac_sample_glines.png)
5. 使用-g:vars选项，即只包括vars调试信息
[![javac_sample_gvars](/resources/2014/11/javac_sample_gvars.png)](/resources/2014/11/javac_sample_gvars.png)
6. 使用-g:source选项，即只包括source调试信息
[![javac_sample_gsource](/resources/2014/11/javac_sample_gsource.png)](/resources/2014/11/javac_sample_gsource.png)

这些调试信息在Eclipse中可在这里设置
[![eclipse_debug_info_settings](/resources/2014/11/eclipse_debug_info_settings.png)](/resources/2014/11/eclipse_debug_info_settings.png)

从JDK中最重要的系统库rt.jar中提取一个class文件，如ThreadLocal，看一下包含哪些调试信息
[![system_rt_library_sample](/resources/2014/11/system_rt_library_sample.png)](/resources/2014/11/system_rt_library_sample.png)
可以看出有lines和source调试信息，但没有vars，这就能解释系统库的方法中局部变量不能获取调试信息，jvm文档中也明确说明了"[LocalVariableTable](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.13)"的作用
> The `LocalVariableTable` attribute is an optional variable-length attribute in the `attributes` table of a `Code` ([§4.7.3](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.3 "4.7.3. The Code Attribute")) attribute. It may be used by debuggers to determine the value of a given local variable during the execution of a method.
> 
> <a name="jvms-4.7.13-110"></a>If `LocalVariableTable` attributes are present in the `attributes` table of a given `Code` attribute, then they may appear in any order. There may be no more than one `LocalVariableTable` attribute per local variable in the `Code`attribute.
既然知道了原因，那怎样解决呢，最直接的方法是下载Java所有源码，然后使用-g选项编译，但我在网上查阅很多资料，最终的结论都是Oracle JDK的源码并没有公开，但OpenJDK（其开发维护人员也是Oracle的）是公开的，并且Oracle JDK是基于OpenJDK，和OpenJDK几乎相同，只是添加了一些不开源的商业库以及虚拟机部分的代码可能和OpenJDK的有些差异。
[![oracle_jdk_and_openjdk](/resources/2014/11/oracle_jdk_and_openjdk.png)
](/resources/2014/11/oracle_jdk_and_openjdk.png "http://openjdk.java.net/faq/")[![oracle_jdk_and_openjdk_2](/resources/2014/11/oracle_jdk_and_openjdk_2.png)](/resources/2014/11/oracle_jdk_and_openjdk_2.png "http://stackoverflow.com/questions/17360011/technically-what-is-the-main-difference-between-oracle-jdk-and-open-jdk")

[![oracle_jdk_differs_from_openjdk](/resources/2014/11/oracle_jdk_differs_from_openjdk.png)
](/resources/2014/11/oracle_jdk_differs_from_openjdk.png "http://stackoverflow.com/questions/22358071/differences-between-oracle-jdk-and-open-jdk-and-garbage-collection")

所以想要具有完整调试信息的rt.jar，可以直接编译openjdk，但更好的方式是重新编译Oracle JDK自带的src.zip，然后和原来的rt.jar合并，可参考[Enabling debugging inside JRE classes](http://www.javalobby.org/java/forums/t103334.html)。

 

那带有完整调试信息的类相比没有（完整）调试信息的类运行时有没有影响？
经过查阅资料，得出的结论是，影响甚微，Java主要的优化是代码优化和运行时环境优化，而不是编译时不添加调试信息。
> [In any language, debugging information is meta information. It by its nature increases the size of the object files, thus increasing load time. During execution outside a debugger, this information is actually completely ignored. As outlined (although not clearly) in the JVM spec the debug information is stored outside the bytecode stream. This means that at execution time there is no difference in the class file. If you want to be sure though, try it out :-).](http://stackoverflow.com/questions/218033/is-there-a-performance-difference-between-javac-debug-on-and-off)
> 
> 
> [Ps. Often for debugging there is value in turning off optimization. That _does_ have a performance impact.](http://stackoverflow.com/questions/218033/is-there-a-performance-difference-between-javac-debug-on-and-off)
 