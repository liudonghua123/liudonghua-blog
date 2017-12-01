---
title: 下载Android源码的痛苦
tags:
  - 7z
  - android
  - disown
  - nohup
  - SimpleHTTPServer
  - tar
  - zip
id: 476
categories:
  - 学习
date: 2014-11-06 20:37:38
---

现在Android 5.0的代码已经推送到AOSP上了，Nexus 5的ROM因为Android 5.0中出现bug也要等到11月12号才发布，为了迫切的定制自己的ROM，所以就开始下载源码了。<!--more-->
直连肯定是不行的，通过goagent或freegate代理下载了好几天都由于国内网络问题经常会出现SSL连接错误（如"[gnutls_handshake() failed](http://stackoverflow.com/questions/13524242/error-gnutls-handshake-failed-git-repository)"，试了几十次"repo sync -j4"都不行），真是泪如泉涌啊。愤怒了，果断用AWS的EC2下载（可参考之前[文章](http://202.203.209.55:8080/?p=6)），速度10+MB/s，20多分钟就下好了，一气呵成，中间没有任何错误，但免费的EC2只有30G磁盘空间，下载完解压到后面就提示空间不够了，看一下.repo文件大概是16G，没办法，只能"分卷压缩-下载-暂停压缩-删除已下载-继续压缩-继续下载-..."，直到完成。

[toc]

##### 分卷压缩/解压缩

[shell gutter="false"]

#方法一
# 最大压缩 (tar+bzip2+7z)
$tar -cf - .repo | 7z a -si -t7z -m0=lzma -mx=9 -mfb=64 -md=32m -ms=on -v1g android_repo.7z
android_repo.7z.001
android_repo.7z.002
...
# 解压缩 (只需要解压第一个)
$7z e android_repo.7z.001
$tar -xf android_repo

#方法二 (tar+bzip+split)
$tar cvjf - .repo | split -b 1g - android_repo.tar.bz2.
或
$tar cvjf - .repo | split --bytes=1GB - android_repo.tar.bz2.
android_repo.tar.bz2.aa.aa
android_repo.tar.bz2.aa.ab
...
# 解压缩 (需要合并后再解压)
$cat android_repo.tar.bz2.a* | tar -xjpvf -

#方法三  (tar+bzip2+zip)
$tar cvjf - .repo | zip -s 1m android_repo.tar.bz2.zip -
android_repo.tar.bz2.z01
android_repo.tar.bz2.zip
......
# 解压缩 (略显麻烦)
$zip -FF android_repo.tar.bz2.zip --out full.zip
$file full.zip
full.zip: Zip archive data, at least v3.0 to extract
$ unzip full.zip
Archive: full.zip
replace -? [y]es, [n]o, [A]ll, [N]one, [r]ename: r
new name: full.tar.bz2
 inflating: full.tar.bz2
$file full.tar.bz2
full.tar.bz2: bzip2 compressed data, block size = 900k
$tar -xjvf full.tar.bz2

[/shell]

##### 在当前目录运行HTTP服务器

[shell gutter="false"]
python -m SimpleHTTPServer &amp;
#或
ruby -run -e httpd . -p 9090 &amp;
[/shell]

后台运行的程序，如果终端关闭，属于这个终端的程序也会停止，所以就需要使用nohup、bash自带的disown或screen
1\. nohup在linux下不能针对已经运行的程序通过"nohup -p pid"让程序脱离终端，只能在启动程序时就调用，如"nohup python -m SimpleHTTPServer &amp;"
2\. [disown](http://www.kossboss.com/linux---move-running-to-process-nohup)的用法是，先让程序后台运行(前台运行时Ctrl+Z)，然后jobs查看id，最后disown -h %id
3\. screen就更简单screen -S name，建立一个可再次连接的会话，然后再这个会话中操作，连接断了重连事直接screen -x name也可以连上

##### 暂停和继续程序的运行

[shell gutter="false"]
kill -s STOP pid
kill -s CONT pid
[/shell]

##### rm文件没有释放硬盘空间

有时候删除一些文件，但是df查看时硬盘空间没有释放，正常情况下要等对应程序退出后才会释放。但可以参考[这里](http://serverfault.com/questions/232525/df-in-linux-not-showing-correct-free-space-after-file-removal#answer-573830)提到的方法rm后立刻释放，一下是我的操作示例。

[shell gutter="false" highlight="26,27,40,41,47,48,69"]
ubuntu@ip-172-31-7-84:~$ ls -li
total 8512136
405120 -rw-rw-r-- 1 ubuntu ubuntu 54 Nov 7 01:50 android_repo.7z.001.md5
405121 -rw-rw-r-- 1 ubuntu ubuntu 54 Nov 7 01:50 android_repo.7z.002.md5
405105 -rw-rw-r-- 1 ubuntu ubuntu 1073741824 Nov 6 09:42 android_repo.7z.003
405104 -rw-rw-r-- 1 ubuntu ubuntu 54 Nov 7 01:51 android_repo.7z.003.md5
405106 -rw-rw-r-- 1 ubuntu ubuntu 1073741824 Nov 6 09:50 android_repo.7z.004
405113 -rw-rw-r-- 1 ubuntu ubuntu 54 Nov 7 01:44 android_repo.7z.004.md5
405107 -rw-rw-r-- 1 ubuntu ubuntu 1073741824 Nov 6 09:57 android_repo.7z.005
405115 -rw-rw-r-- 1 ubuntu ubuntu 54 Nov 7 01:48 android_repo.7z.005.md5
405108 -rw-rw-r-- 1 ubuntu ubuntu 1073741824 Nov 6 10:05 android_repo.7z.006
405116 -rw-rw-r-- 1 ubuntu ubuntu 54 Nov 7 01:48 android_repo.7z.006.md5
405109 -rw-rw-r-- 1 ubuntu ubuntu 1073741824 Nov 6 10:12 android_repo.7z.007
405117 -rw-rw-r-- 1 ubuntu ubuntu 54 Nov 7 01:48 android_repo.7z.007.md5
405110 -rw-rw-r-- 1 ubuntu ubuntu 1073741824 Nov 6 10:20 android_repo.7z.008
405118 -rw-rw-r-- 1 ubuntu ubuntu 54 Nov 7 01:49 android_repo.7z.008.md5
405111 -rw-rw-r-- 1 ubuntu ubuntu 1073741824 Nov 6 10:27 android_repo.7z.009
405119 -rw-rw-r-- 1 ubuntu ubuntu 54 Nov 7 01:49 android_repo.7z.009.md5
405112 -rw-rw-r-- 1 ubuntu ubuntu 1073741824 Nov 7 01:54 android_repo.7z.010
405122 -rw-rw-r-- 1 ubuntu ubuntu 126418976 Nov 7 01:55 android_repo.7z.011
ubuntu@ip-172-31-7-84:~$ lsof |grep deleted
tar 31736 ubuntu 0u CHR 136,2 0t0 5 /dev/pts/2 (deleted)
tar 31736 ubuntu 2u CHR 136,2 0t0 5 /dev/pts/2 (deleted)
7z 31737 ubuntu 1u CHR 136,2 0t0 5 /dev/pts/2 (deleted)
7z 31737 ubuntu 2u CHR 136,2 0t0 5 /dev/pts/2 (deleted)
7z 31737 ubuntu 3w REG 202,1 1073741824 405041 /home/ubuntu/android_repo.7z.001 (deleted)
7z 31737 ubuntu 4w REG 202,1 1073741824 405040 /home/ubuntu/android_repo.7z.002 (deleted)
ubuntu@ip-172-31-7-84:~$ cd /proc/31737/fd
ubuntu@ip-172-31-7-84:/proc/31737/fd$ ll
total 0
dr-x------ 2 ubuntu ubuntu 0 Nov 6 09:19 ./
dr-xr-xr-x 9 ubuntu ubuntu 0 Nov 6 09:19 ../
lr-x------ 1 ubuntu ubuntu 64 Nov 7 01:58 0 -&gt; pipe:[722726]
lrwx------ 1 ubuntu ubuntu 64 Nov 7 01:58 1 -&gt; /dev/pts/2 (deleted)
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 10 -&gt; /home/ubuntu/android_repo.7z.008
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 11 -&gt; /home/ubuntu/android_repo.7z.009
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 12 -&gt; /home/ubuntu/android_repo.7z.010
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 13 -&gt; /home/ubuntu/android_repo.7z.011
lrwx------ 1 ubuntu ubuntu 64 Nov 6 09:19 2 -&gt; /dev/pts/2 (deleted)
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 3 -&gt; /home/ubuntu/android_repo.7z.001 (deleted)
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 4 -&gt; /home/ubuntu/android_repo.7z.002 (deleted)
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 5 -&gt; /home/ubuntu/android_repo.7z.003
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 6 -&gt; /home/ubuntu/android_repo.7z.004
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 7 -&gt; /home/ubuntu/android_repo.7z.005
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 8 -&gt; /home/ubuntu/android_repo.7z.006
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 9 -&gt; /home/ubuntu/android_repo.7z.007
ubuntu@ip-172-31-7-84:/proc/31737/fd$ &gt; 3
ubuntu@ip-172-31-7-84:/proc/31737/fd$ &gt; 4
ubuntu@ip-172-31-7-84:/proc/31737/fd$ ll
total 0
dr-x------ 2 ubuntu ubuntu 0 Nov 6 09:19 ./
dr-xr-xr-x 9 ubuntu ubuntu 0 Nov 6 09:19 ../
lr-x------ 1 ubuntu ubuntu 64 Nov 7 01:58 0 -&gt; pipe:[722726]
lrwx------ 1 ubuntu ubuntu 64 Nov 7 01:58 1 -&gt; /dev/pts/2 (deleted)
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 10 -&gt; /home/ubuntu/android_repo.7z.008
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 11 -&gt; /home/ubuntu/android_repo.7z.009
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 12 -&gt; /home/ubuntu/android_repo.7z.010
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 13 -&gt; /home/ubuntu/android_repo.7z.011
lrwx------ 1 ubuntu ubuntu 64 Nov 6 09:19 2 -&gt; /dev/pts/2 (deleted)
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 3 -&gt; /home/ubuntu/android_repo.7z.001 (deleted)
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 4 -&gt; /home/ubuntu/android_repo.7z.002 (deleted)
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 5 -&gt; /home/ubuntu/android_repo.7z.003
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 6 -&gt; /home/ubuntu/android_repo.7z.004
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 7 -&gt; /home/ubuntu/android_repo.7z.005
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 8 -&gt; /home/ubuntu/android_repo.7z.006
l-wx------ 1 ubuntu ubuntu 64 Nov 7 01:58 9 -&gt; /home/ubuntu/android_repo.7z.007
ubuntu@ip-172-31-7-84:/proc/31737/fd$ df -h
Filesystem Size Used Avail Use% Mounted on
/dev/xvda1 30G 26G 2.7G 91% /
none 4.0K 0 4.0K 0% /sys/fs/cgroup
udev 492M 12K 492M 1% /dev
tmpfs 100M 336K 99M 1% /run
none 5.0M 0 5.0M 0% /run/lock
none 497M 0 497M 0% /run/shm
none 100M 0 100M 0% /run/user
ubuntu@ip-172-31-7-84:/proc/31737/fd$
[/shell]

##### 生成文件checksum的脚本

遍历当前目录下满足文件大小的文件，生成对应的md5、sha

[shell gutter="false"]
for file in `ls`
do
  if [ $(stat -c%s $file) -eq $(echo &quot;1024*1024*1024&quot;|bc) ]
  then
    md5sum $file &gt; &quot;$file.md5&quot;
    shasum $file &gt; &quot;$file.sha&quot;
    cat &quot;$file.md5&quot;
    cat &quot;$file.sha&quot;
  fi
done
[/shell]