---
title: 一个无比纠结的Windows最长路径问题
tags:
  - graphgist
  - neo4j
id: 655
categories:
  - 学习
date: 2014-12-24 10:28:54
---

最近学习[neo4j](http://neo4j.com/)，确实是一个不错的图形数据库，而且[gist](http://gist.neo4j.org/)是一个非常不错的在线演示系统，由于是开源的，所以就想搭建一个本地的gist环境，然后使用本地的neo4j作为后台数据库。

<!--more-->

但按照gist的[README文档](https://github.com/neo4j-contrib/graphgist)，在Windows环境下操作，执行到"bower install"就遇到一个无比烦恼的问题，提示如下错误

```shell
Checking connectivity... done.
fatal: cannot create directory at 'node_modules/grunt-contrib-uglify/node_modules/maxmin/node_modules/gzip-size/node_modules/concat-stream/node_modules/readable-stream/node_modules/core-util-is': File name too long
warning: Clone succeeded, but checkout failed.
You can inspect what was checked out with 'git status'
and retry the checkout with 'git checkout -f HEAD'
```

后来在bower.json中一个一个去掉测试发现是由于安装"jquery-mutate"这个依赖导致的问题，单独在一个非常短路径下执行"bower install jquery-mutate"也会提示如下错误，后来找到"jquery-mutate"的github主页，直接clone是可以的。

所以我的解决方法是先去除/bower.json中"[jquery-mutate](http://www.jqui.net/jquery-projects/jquery-mutate-official/)"依赖，再执行"bower install"，然后到/app/vendor下执行"git clone https://github.com/valtido/jQuery-mutate.git"

官方还有一个[graphgist-cms](https://github.com/neo4j-contrib/graphgist-cms)，对应的演示网站是[graphgist](http://graphgist.neo4j.com/#!/gists)，很容易和前面的gist（网站名称也是graphgist）混淆

这里为什么"bower install jquery-mutate"会出错没有深究，并且解决这个问题也没有多少意义，其根本原因是"万恶、愚蠢的"Windows系统（曾经的、现在的，可能将来的）的explorer、cmd等只能支持最长260字符，弄了一天也没能突破这个限制，官方文档上说可以用UNC路径，即例如"\\.\C:\Windows\"这样的路径可以支持，但也说会存在一些问题，也不支持相对路径，并且连Windows系统的Explorer、CMD等核心组件都不支持，还让第三方程序去支持。

参考资料
1. [NamingFiles, Paths, and Namespaces](http://msdn.microsoft.com/en-us/library/aa365247(VS.85).aspx#maxpath)
2. [WindowsRT 8.1, Windows 8.1, and Windows Server 2012 R2 update: April 2014](http://support.microsoft.com/kb/2919355)
3. [Windows8.1 Update for x64-based Systems (KB2919355)](http://www.microsoft.com/en-us/download/details.aspx?id=42335)
4. [Howto deal with “CMD does not support UNC paths as current directories“](http://mypkb.wordpress.com/2007/02/01/how-to-deal-with-cmd-does-not-support-unc-paths-as-current-directories/)
5. [HANDLING “PATH TOO LONG” ON WINDOWS](http://blog.bfitz.us/?p=65)
6. [gist](http://gist.neo4j.org/)
7. [graphgist](http://graphgist.neo4j.com/#!/gists)
8. [jQuery-mutate](https://github.com/valtido/jQuery-mutate)