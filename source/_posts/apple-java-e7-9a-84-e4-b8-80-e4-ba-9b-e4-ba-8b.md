---
title: Apple Java与的Jetbrains的那些事
tags:
  - java
  - Jetbrains
  - PhpStorm
id: 62
categories:
  - 学习
date: 2014-10-10 09:31:29
---

最近升级了yosemite系统导致原来安装的jetbrains系列开发工具（如Intellj、PhpStorm、RubyMine、PyCharm、WebStorm）无法使用，直接闪退，没有任何错误信息，但查看本机安装的JDK已经有JDK7和JDK8了，JAVA_HOME环境变量等都是配置正确的，Eclipse也是可以使用的，后来运行这个才发现问题<!--more-->

[bash]
/Applications/PhpStorm.app/Contents/MacOS/phpstorm
[JavaAppLauncher Error] CFBundleCopyResourceURL() failed loading MRJApp.properties file
[JavaAppLauncher Error] CFBundleCopyResourceURL() failed while getting Resource/Java directory
JavaVM: Failed to load JVM: /Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home/bundle/Libraries/libserver.dylib
JavaVM: Failed to load JVM: /Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home/bundle/Libraries/libserver.dylib
JavaVM FATAL: Failed to load the jvm library.
[JavaAppLauncher Error] JNI_CreateJavaVM() failed, error: -1
[/bash]

查看phpstorm的[System Requirements and Installation](https://www.jetbrains.com/phpstorm/webhelp/system-requirements-and-installation.html)才发现Mac下只支持Apple的JDK6，不过[这里](https://intellij-support.jetbrains.com/entries/23455956-Selecting-the-JDK-version-the-IDE-will-run-under)提到可以尝试一下JDK7的方法

知道了问题所在就方便解决了
首先安装最新的Apple JDK [JavaForOSX2014-001.dmg
](http://support.apple.com/kb/DL1572)然后设置系统默认的JDK版本
这里的关键是/usr/libexec/java_home

未安装apple的JDK，只安装Oracle JDK（Oracle知道JDK7才开始提供Mac版本，从此Apple JDK就不再开发新版本了）时/usr/libexec/java_home是一个文件，执行

[bash]
/usr/libexec/java_home -V
Matching Java Virtual Machines (2):
1.8.0_20, x86_64: &quot;Java SE 8&quot; /Library/Java/JavaVirtualMachines/jdk1.8.0_20.jdk/Contents/Home
1.7.0_45, x86_64: &quot;Java SE 7&quot; /Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home
[/bash]

安装apple的JDK后/usr/libexec/java_home是一个软链接，指向/System/Library/Frameworks/JavaVM.framework/Versions/Current/Commands/java_home

[bash]
/System/Library/Frameworks/JavaVM.framework/Versions/Current/Commands/java_home -V
Matching Java Virtual Machines (4):
1.8.0_20, x86_64: &quot;Java SE 8&quot; /Library/Java/JavaVirtualMachines/jdk1.8.0_20.jdk/Contents/Home
1.7.0_45, x86_64: &quot;Java SE 7&quot; /Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home
1.6.0_65-b14-466.1, x86_64: &quot;Java SE 6&quot; /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
1.6.0_65-b14-466.1, i386: &quot;Java SE 6&quot; /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
[/bash]

由此可知Mac下Apple JDK安装在/System/Library/Java/JavaVirtualMachines/下，Oracle JDK安装在/Library/Java/JavaVirtualMachines/

值得注意的还有/System/Library/Frameworks/JavaVM.framework/Versions/列出了当前所有的JDK版本，由于我是现有Oracle JDK在安装Apple JDK的，所以这里没有1.7、1.8

[bash]
ll /System/Library/Frameworks/JavaVM.framework/Versions/

total 64
drwxr-xr-x 11 root wheel 374 10 10 08:17 .
drwxr-xr-x 12 root wheel 408 10 10 08:17 ..
lrwxr-xr-x 1 root wheel 10 10 10 08:17 1.4 -&gt; CurrentJDK
lrwxr-xr-x 1 root wheel 10 10 10 08:17 1.4.2 -&gt; CurrentJDK
lrwxr-xr-x 1 root wheel 10 10 10 08:17 1.5 -&gt; CurrentJDK
lrwxr-xr-x 1 root wheel 10 10 10 08:17 1.5.0 -&gt; CurrentJDK
lrwxr-xr-x 1 root wheel 10 10 10 08:17 1.6 -&gt; CurrentJDK
lrwxr-xr-x 1 root wheel 10 10 10 08:17 1.6.0 -&gt; CurrentJDK
drwxr-xr-x 8 root wheel 272 10 10 08:17 A
lrwxr-xr-x 1 root wheel 1 10 10 08:17 Current -&gt; A
lrwxr-xr-x 1 root wheel 59 10 10 08:17 CurrentJDK -&gt; /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents
[/bash]

配置系统默认的JDK版本方法如下备份java_home_bak，修改java_home内容为echo `/usr/libexec/java_home_bak -v 1.6*`

[bash]
sudo mv /usr/libexec/java_home /usr/libexec/java_home_bak
sudo vi /usr/libexec/java_home
echo `/usr/libexec/java_home_bak -v 1.6*`
sudo chmod a+x /usr/libexec/java_home
[/bash]

这时执行
/usr/libexec/java_home返回
/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home

至此jetbrains系列工具就可以使用了，不需要重启或注销，这似乎可以看出jetbrains产品只依赖于/usr/libexec/java_home，JAVA_HOME可以指向其他版本的JDK

如果需要在终端中临时改变JDK版本，可以添加下列Bash脚本到~/.bash_profile中

[bash]

function setjdk() {
if [ $# -ne 0 ]; then
removeFromPath '/System/Library/Frameworks/JavaVM.framework/Home/bin'
if [ -n &quot;${JAVA_HOME+x}&quot; ]; then
removeFromPath $JAVA_HOME
fi
export JAVA_HOME=`/usr/libexec/java_home_bak -v $@`
export PATH=$JAVA_HOME/bin:$PATH
fi
}

function removeFromPath() {
export PATH=$(echo $PATH | sed -E -e &quot;s;:$1;;&quot; -e &quot;s;$1:?;;&quot;)
}

[/bash]

以后切换时就只需要setjdk 1.6或setjdk 1.7等等的了