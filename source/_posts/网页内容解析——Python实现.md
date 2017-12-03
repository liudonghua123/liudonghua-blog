---
title: 网页内容解析——Python实现
tags:
  - pyQuery
  - python
id: 592
categories:
  - 学习
date: 2014-12-02 15:16:28
---

网页的内容一般是html，是纯文本格式的，有特定的文档结构。如果需要通过提取一些结构相似的网页的特定信息，一般通过程序编程实现。

<!--more-->

当然程序实现由很多种方式，有用常用的C/C++/Java等语言，但这些语言写的虽然执行效率高，但需要写很多代码，开发效率不高，而且代码越多意味着可能存在的bug也就越多，所以对于简单网页解析一般通过一些脚本语言来实现，如Python、Ruby，甚至Javascript（如果需求简单，不涉及读写文件、数据库等，用JS还更好一些，毕竟JS与html"天然的"最好搭档）

这篇文章就简单介绍一些实用Python来实现（原本想要JS，但涉及跨域访问的问题，后面想尝试用浏览器扩张方式实现）从一个（一组）特定的网页中提取特定的信息

解析网页的过程一般分为
1.根据网址获取HTML
2.解析HTML获取特定信息

第一步根据网址获取HTML这就不多说了，查阅Python文档，使用urllib2来实现，大致代码如下

```python
import urllib2

request = urllib2.Request('http://item.jd.com/1164932931.html')
response = urllib2.urlopen(request)
html = response.read()

# decode the html content using the charset of HTTP response Content-Type header
encoding = response.headers['content-type'].split('charset=')[-1]
html_decoded = html.decode(encoding)
```

第二步解析HTML获取网页特定信息，这里有很多实现，如使用自带的库HTMLParse（[2.x](https://docs.python.org/2/library/htmlparser.html), [3.x](https://docs.python.org/3/library/html.parser.html)），但这种方式类似于Java中的基于事件的SAX解析方式，需要些很多代码，我想使用DOM方式解析出整颗树，然后利用类似于jQuery中的ID、Class、Tag选择器快捷查找解析功能（如果可能），后来找到有很多这样的库，如[BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/bs4/doc/)、[PyQuery](https://pypi.python.org/pypi/pyquery)，这里用PyQuery来实现，实现的代码如下

```python
from pyquery import PyQuery as pq
import HTMLParser

# 类似于jQuery中的$('html')获取整棵文档树
# 这里可用未解码的html或解码或的html_decoded，效果一样
pqHtml = pq(html)
htmlParser = HTMLParser.HTMLParser()
# 这里解决解码问题，如果不使用htmlParser.unescape，中文的输出会是如"&#23478;&#20855;"这种信息，不是UTF-8、Unicode、GBK的编码，不能使用str.decode(encoding_name)方式解码
print htmlParser.unescape(str(pqHtml('div.breadcrumb a')))
```

其实PyQuery 就自带了GET/POST等资源获取方式，以上两部分可以简化为两三行代码（不包括import）

```python
from pyquery import PyQuery as pq
import HTMLParser

pqHtml=pq(url='http://item.jd.com/1164932931.html')
htmlParser = HTMLParser.HTMLParser()
print htmlParser.unescape(str(pqHtml('div.breadcrumb a')))
```

这样就可以解析出如下一个特定网页中的特定信息(后续还要一点点处理)
[![jd_html_example](/resources/2014/12/jd_html_example.png)](/resources/2014/12/jd_html_example.png)
代码输出为

```shell
> python AnalysisJD.py
<a href="http://channel.jd.com/furniture.html">家具</a><a href="http://channel.jd.com/9847-9850.html">餐厅家具</a> > <a href="http://list.jd.com/9847-9850-9877.html">餐桌</a> > <a href="http://www.jd.com/pinpai/9877-53848.html">杰希家具（J&C furniture）</a> > <a href="http://item.jd.com/1164932931.html">杰希家具北欧简约餐桌 时尚设计 大小餐客...</a>
```

最后再提一下Windows下安装PyQuery过程中遇到的问题
首先直接运行"pip install pyquery"提示如下错误

```shell
running build_ext

building 'lxml.etree' extension

C:\Python27\lib\distutils\dist.py:267: UserWarning: Unknown distribution option: 'bugtrack_url'

  warnings.warn(msg)

error: Unable to find vcvarsall.bat
```

查阅资料因为Python 2.7默认使用VS2008对源码库编译安装，所以可以重新定义VS90COMNTOOLS环境变量
> For Windows installations:
> 
> 
> While running setup.py for package installations, Python 2.7 searches for an installed Visual Studio 2008.You can trick Python to use a newer Visual Studio by setting the correct path in `VS90COMNTOOLS`environment variable before calling `setup.py`.
> 
> 
> Execute the following command based on the version of Visual Studio installed:
> 
> 
> *   Visual Studio 2010 (VS10): `SET VS90COMNTOOLS=%VS100COMNTOOLS%`
> *   Visual Studio 2012 (VS11): `SET VS90COMNTOOLS=%VS110COMNTOOLS%`
> *   Visual Studio 2013 (VS12): `SET VS90COMNTOOLS=%VS120COMNTOOLS%`
我电脑上装的是2012，如下设置

```shell
C:\Users\Liu.D.H>rem 检测是否有VS120COMNTOOLS环境变量
C:\Users\Liu.D.H>echo %VS120COMNTOOLS%
C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\Tools\
C:\Users\Liu.D.H>rem 重新定义VS90COMNTOOLS环境变量，值为VS120COMNTOOLS
C:\Users\Liu.D.H>SET VS90COMNTOOLS=%VS120COMNTOOLS%
```

然后继续执行pip，报以下错误

```shell
c:\users\liud~1.h\appdata\local\temp\pip_build_Liu.D.H\lxml\src\lxml\includes\etree_defs.h(14) : fatal error C1083: Cannot open include file: 'libxml/xmlversion.h': No such file or directory

C:\Python27\lib\distutils\dist.py:267: UserWarning: Unknown distribution option: 'bugtrack_url'

  warnings.warn(msg)

error: command 'C:\\Program Files (x86)\\Microsoft Visual Studio 12.0\\VC\\BIN\\amd64\\cl.exe' failed with exit status 2
```

很明显又是libxml这个库找不到，搜索到[libxml-python](http://users.skynet.be/sbi/libxml-python/)、[lxml](http://www.lfd.uci.edu/~gohlke/pythonlibs/#lxml)
安装过程中提示"Python version 2.7 required, which was not found in the registry"
查看注册表项只有32位的HKEY_LOCAL_MACHINE\SOFTWARE\Python\PythonCore\2.7\InstallPath，而没有64位的HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Python\PythonCore\2.7\InstallPath（我电脑上装的是ActiveState Python 2.7.8 64位，没研究知道为什么注册表项不对，安装官方版本似乎没有这个问题），所以添加64位的注册表项，仿照32位的设置
安装完前一个还是没用，后一个才可以解决以上头文件找不到问题。
至此终于安装好pyQuery及其依赖的库，不得不吐槽，太繁琐了，还是Linux或者Mac系统方便，如在Ubuntu下运行以下命令即可

```shell
sudo apt-get install libxml2-dev libxslt1-dev python-dev
sudo pip install pyquery
```

参考资料
1. [HOWTO Fetch Internet Resources Using urllib2](https://docs.python.org/2/howto/urllib2.html)
2.[pyquery](https://pypi.python.org/pypi/pyquery/1.2.1)
3.[parsing-html-python](http://stackoverflow.com/questions/11709079/parsing-html-python)
4.[installing-pyquery-via-pip](http://stackoverflow.com/questions/21489720/installing-pyquery-via-pip)
5.[python-version-2-7-required-which-was-not-found-in-the-registry](https://avaminzhang.wordpress.com/2011/11/24/python-version-2-7-required-which-was-not-found-in-the-registry/)
6.[what-is-the-difference-between-sax-and-dom](http://stackoverflow.com/questions/6828703/what-is-the-difference-between-sax-and-dom)