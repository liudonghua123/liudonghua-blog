---
title: 极速下载Android源码
tags:
  - android
id: 6
comment: false
categories:
  - 学习
date: 2014-09-28 19:47:05
---

或许大家都有感觉，国内因为google被屏蔽的原因，下载Android源码是一件非常痛苦的事情，不仅速度慢而且下载过程中有时候会网络中断，要想一次性完整下载几乎很难，最近看到一片文章提到可以免费使用一年Amazon的EC2云服务器，今年尝试了一下，还是非常不错，相比阿里云的ECS感觉一个明显的优点是官方或社区提供了非常多的AMI（有些类似于安装有特定软件的系统镜像），如想搭建一个wordpress可以很快找到一个配置好的服务器环境，然后就可以直接使用了。<!--more-->

今天测试了一下，下载Android master分支源码，下载完成后总共28G（其实就下载了13G，即.repo目录的大小），总共用了1931s，大概就半小时。

其中用到的一些命令如下

使用screen保持远程连接会话，防止终端连接中断后下载进程也停止
# 创建一个名称为android的会话
screen -S android
# 查看目前已有的会话
screen -ls
# 恢复名称为android的会话连接（终端连接中断中使用）
screen -x android

# 安装Oracle的JDK
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java6-installer
（如果要安装JDK7、8可以把oracle-java6-installer替换为oracle-java7-installer、oracle-java8-installer）

# 安装一些依赖库，适合ubuntu 14.04
sudo apt-get install git gnupg flex bison gperf build-essential \
zip curl libc6-dev libncurses5-dev x11proto-core-dev \
libx11-dev libreadline6-dev libgl1-mesa-glx \
libgl1-mesa-dev g++-multilib mingw32 tofrodos \
python-markdown libxml2-utils xsltproc zlib1g-dev

mkdir ~/bin
PATH=~/bin:$PATH

curl https://storage.googleapis.com/git-repo-downloads/repo &gt; ~/bin/repo
chmod a+x ~/bin/repo

mkdir android
cd android

# 使用文本浏览器获取.netrc内容
w3m https://android.googlesource.com/new-password

repo init -u https://android.googlesource.com/a/platform/manifest

# 记录下载Android源码日志（通过tee同时写到标准输出和文件）
repo sync -j2 2&gt;&amp;1 | tee ~/android_source_sync.log

以下命令计算上一条命令的运行时间
start_time=`date +%s` \
&amp;&amp; repo sync -j2 2&gt;&amp;1 | tee ~/android_source_sync.log \
&amp;&amp; end_time=`date +%s` \
&amp;&amp; echo execution time was `expr $end_time - $start_time` s.