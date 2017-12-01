---
title: mac下的awk、sed
tags:
  - bash
  - mac
id: 53
categories:
  - 学习
date: 2014-10-09 09:39:47
---

linux/unix下的awk、sed是常用的功能强大的文本处理工具，但mac中的这些工具与普通的linux不同。<!--more-->
例如想以":"分割$PATH中路径，如果使用普通linux可以使用如下命令

1.  <span class="pln">echo $PATH </span><span class="pun">|</span><span class="pln"> sed </span><span class="pun">-</span><span class="pln">e </span><span class="str">'s/:/\n/g'</span>
但如果在mac下就不行了，需要使用以下的几种方式

1.  [<span class="pln">echo $PATH</span><span class="pun">|</span><span class="pln">sed </span><span class="pun">-</span><span class="pln">e </span><span class="str">'s/:/\'$'</span><span class="pln">\n</span><span class="pun">/</span><span class="pln">g</span><span class="str">'
</span>](http://nlfiedler.github.io/2010/12/05/newlines-in-sed-on-mac.html)or

1.  [<span class="pln">echo $PATH</span><span class="pun">|</span><span class="pln">sed </span><span class="pun">-</span><span class="pln">e $</span><span class="str">'s/:/\\\n/g'
</span>](http://stackoverflow.com/questions/10748453/replace-comma-with-newline-in-sed)or

1.  [<span class="pln">echo $PATH</span><span class="pun">|</span><span class="pln">sed </span><span class="pun">-</span><span class="pln">e </span><span class="str">'</span><span class="pln">s</span><span class="pun">/:/</span><span class="pln">\</span>](http://stackoverflow.com/questions/10748453/replace-comma-with-newline-in-sed)
2.  [<span class="pun">/</span><span class="pln">g</span><span class="str">'</span>](http://stackoverflow.com/questions/10748453/replace-comma-with-newline-in-sed)
&nbsp;

如果想在mac下使用gnu的awk、sed可以安装gawk\gnu-sed

1.  <span class="pln">brew install gawk</span>
2.  <span class="pln">brew install gnu</span><span class="pun">-</span><span class="pln">sed</span>