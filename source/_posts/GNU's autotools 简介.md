---
title: GNU's autotools 简介
tags:
  - autoconf
  - automake
  - autotools
  - libtool
id: 399
comment: false
categories:
  - 学习
date: 2014-11-01 16:31:30
---

我自己的放在Github上的bc命令行计算器工程在travis持续集成的编译中总是出现一些问题，经过分析这是由于不正确使用autotools导致的，这就促使我学习一下autotools工具包的原理及使用方式。

<!--more-->

一般我们在Linux下通过源码安装软件一般都是通过一下形式安装的

```shell
./configure && make && make install
```

这就涉及到autotools(主要涉及三个命令autoconf、automake、libtool)

可通过"sudo apt-get install automake"来安装autotools系列工具

```shell
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ aclocal
程序 'aclocal' 已包含在下列软件包中：
 * automake
 * automake1.10
 * automake1.11
 * automake1.4
 * automake1.9
请尝试：sudo apt-get install <选定的软件包>
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ sudo apt-get install automake
正在读取软件包列表... 完成
正在分析软件包的依赖关系树
正在读取状态信息... 完成
将会安装下列额外的软件包：
  autoconf autotools-dev
建议安装的软件包：
  autoconf2.13 autoconf-archive gnu-standards autoconf-doc libtool
下列【新】软件包将被安装：
  autoconf automake autotools-dev
......
```

以下是我[Building a GNU Autotools Project](http://inti.sourceforge.net/tutorial/libinti/autotoolsproject.html)这篇文章的使用autotools实践的一个简单的例子
用到的文件如下

```shell
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ cat configure.ac
AC_INIT(src/main.cc)

PACKAGE=helloworld
VERSION=0.1.0

AM_INIT_AUTOMAKE($PACKAGE, $VERSION)

INTI_REQUIRED_VERSION=1.0.7
#PKG_CHECK_MODULES(INTI, inti-1.0 >= $INTI_REQUIRED_VERSION)
AC_SUBST(INTI_CFLAGS)
AC_SUBST(INTI_LIBS)

AC_PROG_CXX

AC_OUTPUT(Makefile src/Makefile)
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ cat Makefile.am
AUTOMAKE_OPTIONS = foreign
SUBDIRS = src
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ cat src/Makefile.am
bin_PROGRAMS = helloworld

AM_CXXFLAGS = $(INTI_CFLAGS)

helloworld_SOURCES = main.cc helloworld.cc helloworld.h
helloworld_LDADD = $(INTI_LIBS)
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ cat src/helloworld.h
#ifndef _HELLOWORLD_H
#define _HELLOWORLD_H

int max(int a, int b);

#endif
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ cat src/helloworld.cc
#include "helloworld.h"

int max(int a, int b)
{
    return a > b ? a : b;
}
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ cat src/main.cc
#include <iostream>
#include "helloworld.h"

using namespace std;

int main()
{
    int a = 10;
    int b = 5;
    int c = max(a,b);
    cout << "max(a,b):" << c << endl;
}

liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
```

高亮部分为每个阶段生产的新文件，详情注释部分

```shell
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ # 初始的必须文件(configure.ac、Makefile.am，configure.ac可通过autoscan生成configure.scan修改而来)
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ # configure.in已过时，使用configure.ac代替
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ tree
.
├── configure.in
├── Makefile.am
└── src
    ├── helloworld.cc
    ├── helloworld.h
    ├── main.cc
    └── Makefile.am

1 directory, 6 files
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ # 通过执行aclocal生成aclocal.m4（包含configure.ac中用到的宏，如AM_INIT_AUTOMAKE），以及autom4te.cache（用于加速autotools运行的缓存）
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ aclocal
aclocal: warning: autoconf input should be named 'configure.ac', not 'configure.in'
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ mv configure.in configure.ac
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ aclocal
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ tree
.
├── aclocal.m4
├── autom4te.cache
│   ├── output.0
│   ├── output.1
│   ├── requests
│   ├── traces.0
│   └── traces.1
├── configure.ac
├── Makefile.am
└── src
    ├── helloworld.cc
    ├── helloworld.h
    ├── main.cc
    └── Makefile.am

2 directories, 12 files
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ ll | wc -l
8
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ ll
总用量 64
drwxrwxr-x 4 liudonghua liudonghua  4096 11月  1 15:33 ./
drwxrwxr-x 5 liudonghua liudonghua  4096 11月  1 15:32 ../
-rw-rw-r-- 1 liudonghua liudonghua 39670 11月  1 15:33 aclocal.m4
drwxr-xr-x 2 liudonghua liudonghua  4096 11月  1 15:33 autom4te.cache/
-rw-rw-r-- 1 liudonghua liudonghua   272 11月  1 13:28 configure.ac
-rw-rw-r-- 1 liudonghua liudonghua    41 11月  1 14:31 Makefile.am
drwxrwxr-x 2 liudonghua liudonghua  4096 11月  1 15:30 src/
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ # automake --add-missing（简写成automake -a）用于生成一些autotools运行必要的脚本（其实是链接到automake安装目录下），如depcomp、install-sh、missing、Makefile.in（需要Makefile.am）
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ automake --add-missing
configure.ac:6: warning: AM_INIT_AUTOMAKE: two- and three-arguments forms are deprecated.  For more info, see:
configure.ac:6: http://www.gnu.org/software/automake/manual/automake.html#Modernize-AM_005fINIT_005fAUTOMAKE-invocation
configure.ac:6: installing './install-sh'
configure.ac:6: installing './missing'
src/Makefile.am: installing './depcomp'
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ ll | wc -l
12
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ ll
总用量 88
drwxrwxr-x 4 liudonghua liudonghua  4096 11月  1 15:35 ./
drwxrwxr-x 5 liudonghua liudonghua  4096 11月  1 15:32 ../
-rw-rw-r-- 1 liudonghua liudonghua 39670 11月  1 15:33 aclocal.m4
drwxr-xr-x 2 liudonghua liudonghua  4096 11月  1 15:35 autom4te.cache/
-rw-rw-r-- 1 liudonghua liudonghua   272 11月  1 13:28 configure.ac
lrwxrwxrwx 1 liudonghua liudonghua    32 11月  1 15:35 depcomp -> /usr/share/automake-1.14/depcomp*
lrwxrwxrwx 1 liudonghua liudonghua    35 11月  1 15:35 install-sh -> /usr/share/automake-1.14/install-sh*
-rw-rw-r-- 1 liudonghua liudonghua    41 11月  1 14:31 Makefile.am
-rw-rw-r-- 1 liudonghua liudonghua 23457 11月  1 15:35 Makefile.in
lrwxrwxrwx 1 liudonghua liudonghua    32 11月  1 15:35 missing -> /usr/share/automake-1.14/missing*
drwxrwxr-x 2 liudonghua liudonghua  4096 11月  1 15:35 src/
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ # autoconf生成最终的configure脚本
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ autoconf
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ ll | wc -l
13
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ ll
总用量 88
drwxrwxr-x 4 liudonghua liudonghua  4096 11月  1 15:35 ./
drwxrwxr-x 5 liudonghua liudonghua  4096 11月  1 15:32 ../
-rw-rw-r-- 1 liudonghua liudonghua 39670 11月  1 15:33 aclocal.m4
drwxr-xr-x 2 liudonghua liudonghua  4096 11月  1 15:35 autom4te.cache/
-rwxrwxr-x 1 liudonghua liudonghua 131248 11月  1 15:36 configure*
-rw-rw-r-- 1 liudonghua liudonghua   272 11月  1 13:28 configure.ac
lrwxrwxrwx 1 liudonghua liudonghua    32 11月  1 15:35 depcomp -> /usr/share/automake-1.14/depcomp*
lrwxrwxrwx 1 liudonghua liudonghua    35 11月  1 15:35 install-sh -> /usr/share/automake-1.14/install-sh*
-rw-rw-r-- 1 liudonghua liudonghua    41 11月  1 14:31 Makefile.am
-rw-rw-r-- 1 liudonghua liudonghua 23457 11月  1 15:35 Makefile.in
lrwxrwxrwx 1 liudonghua liudonghua    32 11月  1 15:35 missing -> /usr/share/automake-1.14/missing*
drwxrwxr-x 2 liudonghua liudonghua  4096 11月  1 15:35 src/
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ # 生成最终的Makefile以及一些日志文件config.log、config.status
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ ./configure
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... no
checking for mawk... mawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
......
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ ll | wc -l
16
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$ ll
总用量 288
drwxrwxr-x 4 liudonghua liudonghua   4096 11月  1 15:37 ./
drwxrwxr-x 5 liudonghua liudonghua   4096 11月  1 15:32 ../
-rw-rw-r-- 1 liudonghua liudonghua  39670 11月  1 15:33 aclocal.m4
drwxr-xr-x 2 liudonghua liudonghua   4096 11月  1 15:35 autom4te.cache/
-rw-rw-r-- 1 liudonghua liudonghua   9035 11月  1 15:37 config.log
-rwxrwxr-x 1 liudonghua liudonghua  28973 11月  1 15:37 config.status*
-rwxrwxr-x 1 liudonghua liudonghua 131248 11月  1 15:36 configure*
-rw-rw-r-- 1 liudonghua liudonghua    272 11月  1 13:28 configure.ac
lrwxrwxrwx 1 liudonghua liudonghua     32 11月  1 15:35 depcomp -> /usr/share/automake-1.14/depcomp*
lrwxrwxrwx 1 liudonghua liudonghua     35 11月  1 15:35 install-sh -> /usr/share/automake-1.14/install-sh*
-rw-rw-r-- 1 liudonghua liudonghua  23727 11月  1 15:37 Makefile
-rw-rw-r-- 1 liudonghua liudonghua     41 11月  1 14:31 Makefile.am
-rw-rw-r-- 1 liudonghua liudonghua  23457 11月  1 15:35 Makefile.in
lrwxrwxrwx 1 liudonghua liudonghua     32 11月  1 15:35 missing -> /usr/share/automake-1.14/missing*
drwxrwxr-x 3 liudonghua liudonghua   4096 11月  1 15:38 src/
liudonghua@liudonghua-Ubuntu:~/tests/helloworld$
```

总结
1\. 使用autotools封装自己的C\C++程序，这样相比直接使用make可以更好的编译检测及屏蔽不同平台的一些环境差异，而且还支持打包、安装、卸载等功能
2\. 使用autotools关键在于写configure.ac（一个）和Makefile.am（可能有多个）
3\. autoconf与automake没有执行先后顺序。autoconf需要输入configure.ac，输出configure；automake需要输入configure.ac和Makefile.am，输出Makfile.in

configure.ac、Makefile.am规则可参考
[Autoconf and Automake Tutorial](http://amjith.blogspot.com/2009/04/autoconf-and-automake-tutorial.html)
[A tutorial for porting to autoconf & automake](http://mij.oltrelinux.com/devel/autoconf-automake/)
[Building a GNU Autotools Project](http://inti.sourceforge.net/tutorial/libinti/autotoolsproject.html)

注：
"A tutorial for porting to autoconf & automake"中的例子必须把src/Makefile.am中的-std=c99去掉或改成-[std=gnu11](https://gcc.gnu.org/onlinedocs/gcc/Standards.html)
"Building a GNU Autotools Project"的configure.am中的"PKG_CHECK_MODULES(INTI, inti-1.0 >= $INTI_REQUIRED_VERSION)"在一些环境下不可用