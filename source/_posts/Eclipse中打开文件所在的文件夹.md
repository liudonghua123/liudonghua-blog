---
title: Eclipse中打开文件所在的文件夹
tags:
  - eclipse
id: 550
categories:
  - 学习
date: 2014-11-20 13:45:39
---

在使用eclipse时，会经常在外面文件浏览器中打开当前所在的文件，虽然可以通过安装插件的方式，但其实有一种更好的方式，即使用External Tool Configuration，添加一项即可。

<!--more-->

[toc]

### Windows

Location: ${env_var:SystemRoot}\explorer.exe
Arguments: /select,${resource_loc}
如下图所示
[![windows_eclipse_open_current_directory](/resources/2014/11/windows_eclipse_open_current_directory.png)](/resources/2014/11/windows_eclipse_open_current_directory.png)

### Mac OS X

[![mac_eclipse_reveal_current_file](/resources/2014/11/mac_eclipse_reveal_current_file.png)](/resources/2014/11/mac_eclipse_reveal_current_file.png)
Location: /usr/bin/open
Arguments: -R "${resource_loc}"
或
Location: /usr/bin/osascript
Arguments: -e "tell application \"Finder\"" -e "reveal POSIX file \"${resource_loc}\"" -e "activate" -e "end tell"

### Linux（Ubuntu）

Location: nautilus
Arguments: ${resource_loc}

参考资料
1. [in-eclipse-reveal-current-file-in-filesystem](http://stackoverflow.com/questions/1161240/in-eclipse-reveal-current-file-in-filesystem)
2. [Open the windows explorer with a file selected in eclipse...](http://www.eclipsezone.com/eclipse/forums/t77655.html)