---
title: 你可能不知道的Java反编译器
tags:
  - decompile
  - java
  - Procyon
id: 540
categories:
  - 学习
date: 2014-11-14 21:55:52
---

我最近看到一个使用动画演示排序算法的Android应用[sorts](https://play.google.com/store/apps/details?id=com.sorts&hl=en)，做的挺不错的，对学习算法还是有很大帮助，但美中不足的是缺少冒泡算法，所以想找一下是否有源码，然后再自己添加进去，最终还是没找到，所以就只有反编译看一下能不能添加。

<!--more-->

首先想到的是强大的APKTool，这个工具可以反编译res资源、AndroidManifest.xml，Java编译后生成的class转换成的dex可以通过内建的baksmali转换成smali文件，但smali文件只适用于小的修改，所以直接使用dex2jar把dex先转换成class文件，再有jd-gui转换成java源文件。虽然可以转换，但转换之后的Java代码有些和源代码相差太大，不好理解，如以下是部分难看懂的代码（要想看懂也是可以的，不过需要很多其他知识，如可以参考一下[difference-between-packed-switch-and-sparse-switch-dalvik-opcode](http://stackoverflow.com/questions/19855800/difference-between-packed-switch-and-sparse-switch-dalvik-opcode)，[dealing-with-labels-in-decompiled-code](http://stackoverflow.com/questions/6347930/dealing-with-labels-in-decompiled-code)等）

```java
for (;;) {
	return;
	((Animation) InsertionActivity.this.animationList.get(j))
			.setStartOffset(600000L + ((Animation) InsertionActivity.this.animationList
					.get(j)).getStartOffset());
	j++;
	break;
	......
```

为了生成更容易理解、更接近源代码的代码，寻找新的Java反编译器就非常必要了，这里简单总结一下一些常用的Java反编译器（有点少，很多都老旧过时的，并且这个领域额也不热门）
<del>1. JAva Decompiler (JAD – No longer maintained)</del>
<del>2. DJ Java Decompiler (UI for JAD)</del>
3. JD Java Decompiler（JD-GUI的核心）
4. [JD-GUI](http://jd.benow.ca/)
5. [Procyon](https://bitbucket.org/mstrobel/procyon)
JAD的只支持近版本，而且非常老旧，只能抛弃
JD用的比较多，效果也还好
Procyon是我认为目前最好的，并且这个项目还在不断开发维护，值得推荐。

并且我用基于Procyon的[SecureTeam](http://www.secureteam.net/Java-Decompiler.aspx)反编译出sorts的Java源代码，做了一些修改，直接又可以当作源码编译。

Procyon反编译后的Java我做的修改有
1. 删除xxx$xxx内部类（这是冗余的，xxx已经包括这些内部类了）
2. 修改一些类名，如View$OnClickListener改为View.OnClickListener

我想这是一些这个反编译器存在的不足，不过相比JD反编译出的质量高了很多，后面有时间可以给Procyon提交一些代码让它能够处理以上两个不足的地方，然后再顺便解决源码里R资源直接是数字的问题，让他替换成R.id.xxx或R.layout.xxx等更有意义的代码

以下是自己写的使用Python替换反编译代码中的R资源

```python
__author__ = 'Liu.D.H'

import re
import fileinput
import argparse
import os
import fnmatch
import sys

def parseRresource(filePath):
    r_resource = {}
    classbeginpattern = re.compile(r".*class\s*(?P<className>[a-zA-Z_][a-zA-Z_0-9]*)\s*{")
    classendpattern = re.compile(r"\s*}\s*")
    fieldpattern = re.compile(r".*int\s*(?P<fieldName>[_A-Za-z][_0-9A-Za-z]*)\s*=\s*(?P<fieldValue>\d+);")
    innerclassname = None
    innerclassfieldname = None
    innerclassfieldvalue = None
    parseinnerclass = False
    with open(filePath) as file:
        for line in file:
            classbegin = classbeginpattern.match(line)
            classend = classendpattern.match(line)
            # Parse begin or end of inner class
            if classbegin:
                parseinnerclass = True
                innerclassname = classbegin.group("className")
                continue
            elif classend:
                parseinnerclass = False
                continue
            # Parse field of inner class
            if parseinnerclass:
                innerclassfield = fieldpattern.match(line)
                if innerclassfield:
                    innerclassfieldname = innerclassfield.group("fieldName")
                    innerclassfieldvalue = innerclassfield.group("fieldValue")
                    r_resource[innerclassfieldvalue] = "R.%s.%s" % (innerclassname, innerclassfieldname)

    return r_resource

# http://stackoverflow.com/questions/7868554/python-re-subs-replace-function-doesnt-accept-extra-arguments-how-to-avoid
def numberrepl(rResource):
    def repl(matchobj):
        number = matchobj.group(0)
        if number in rResource:
            return rResource[number]
        else:
            return number
    return repl

def processJavaFile(filePath, r_resource):
    for line in fileinput.input(filePath, inplace=1):
        line = re.sub("\d+", numberrepl(r_resource), line)
        print line,

def getFilePaths(rootDirectoryPath):
    filePaths = []
    rFilePaths = []

    def recursiveSubFiles(directoryPath):
        global rFilePath
        for root, directories, files in os.walk(directoryPath):
            for filename in files:
                if filename == "R.java":
                    rFilePath = os.path.join(root, filename)
                    rFilePaths.append(rFilePath)
                elif fnmatch.fnmatch(filename, "*.java"):
                    # Join the two strings in order to form the full filepath.
                    filepath = os.path.join(root, filename)
                    filePaths.append(filepath)  # Add it to the list.
            for dirName in directories:
                dirPath = os.path.join(root, dirName)
                recursiveSubFiles(dirName)

    recursiveSubFiles(rootDirectoryPath)

    return filePaths, rFilePaths

def main(argv):
    # parse command line argument
    parser = argparse.ArgumentParser(description='Replace R resource name with its value.')
    parser.add_argument('-s', '--srcRootDir', nargs="?", const=".", default=".", help='the root direcotory of src')
    args = parser.parse_args(argv[1:])
    # get all java source file paths
    javaFilePaths, rFilePaths = getFilePaths(args.srcRootDir)
    # parse R.java resources
    rResources = parseRresource(rFilePaths[0])
    # process all java source file
    for filePath in javaFilePaths:
        processJavaFile(filePath, rResources)

if __name__ == "__main__":
    main(sys.argv)

```