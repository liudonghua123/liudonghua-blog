---
title: Windows中可替代CMD的工具
tags:
  - Babun
  - Clink
  - Cmder
  - Conemu
  - Windows Shell
id: 764
categories:
  - 学习
date: 2015-05-10 10:39:12
---

对于习惯Linux Bash Shell的用户，在Windows下操作，使用cmd会感觉那个东西好笨拙、难用。好在现在有一些工具可以在Windows下实现类似Bash Shell类似的体验。<!--more-->

众所周知的一些工具如[MinGW](http://www.mingw.org/)（[MSYS2](http://sourceforge.net/projects/msys2/)）、[Cygwin](http://www.cygwin.com/)可以满足以上功能，但我不是很推荐，因为安装略微繁琐（需要联网、国内访问慢），而且那个shell窗口的界面和cmd一样，非常难看，缺少个性化配置。

另外还有[Clink](http://mridgers.github.io/clink/)(主要提供Windows的readline库功能，这对于shell中命令自动补齐、命令历史浏览非常重要，并且提供lua扩展)、[conemu](https://code.google.com/p/conemu-maximus5/)、[Cmder](http://gooseberrycreative.com/cmder/)（其实封装了clink、conemu、[msysgit](https://github.com/msysgit/msysgit)）。

最后就是我要重点推荐的[Babun](http://babun.github.io/)。

ps: 在安装之前需要清空Windows下的HOME环境变量（"set HOME="就可以清除了），然后在运行install脚本安装（如install /target "D:\Program_Files\babun-1.1.0"），如果忘了这一点，就会出现类似于[Vim: missing color scheme "ir_black"](https://github.com/babun/babun/issues/78)这样的错误，这时删除.babun重新安装吧！
并且注意安装完成之后也不能恢复Windows下的HOME环境变量，不过可以修改.babun/babun.bat文件，在其中添加如下第10行就可以解决HOME环境变量的问题。所以如果如果安装之前没有清空Windows下的HOME环境变量，也不想重装，那么也可以先创建/home/YOUR_USERNAME目录，然后unset HOME; babun install，最后同样修改.babun/babun.bat文件。
总之就是必须修改.babun/babun.bat重新定义babun中的HOME，否则只有清空Windows中的HOME（有风险，非常不建议）

```shell
@echo off
setlocal enableextensions enabledelayedexpansion

set SCRIPT_PATH=%~dp0
set SCRIPT_PATH=%SCRIPT_PATH:\=/%
set BABUN_HOME=%SCRIPT_PATH%

:BEGIN
set CYGWIN_HOME=%BABUN_HOME%\cygwin
set HOME=%CYGWIN_HOME%\home\Liu.D.H
if exist "%CYGWIN_HOME%\bin\mintty.exe" goto RUN
if not exist "%CYGWIN_HOME%\bin\mintty.exe" goto NOTFOUND
```

当然还有微软自己在.Net时代改进的Shell——[PowerShell](https://technet.microsoft.com/en-us/scriptcenter/powershell.aspx)，但那东西有点奇葩，太依赖于Windows本身的特性，因此至少目前只能在Windows下工作。

下面附一些截图

![](http://conemu-maximus5.googlecode.com/svn/files/ConEmuSplits3.png)

![](http://gooseberrycreative.com/cmder/img/main.jpg)

<video width="100%" controls>
    <source src="/resources/2015/05/Introduction-to-the-Babun-Project-HD.mp4">
</video>

 