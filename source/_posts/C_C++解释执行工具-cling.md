---
title: C_C++解释执行工具-cling
tags:
  - C
  - C++
  - cling
  - ch
  - tcc
  - picoc
  - cint
  - iGCC
  - ccons
  - CompCert
id: 000
categories:
  - 学习
date: 2017-12-29 10:18:00
---

为了方便学习C/C++，今天突然想到如果有一个类似于 Python、Node、Ruby 这些脚本语言的交互式的环境可以使用，那应该是非常棒的学习C/C++体验，特别适用于初学者。

现在 Java 9 都提供了这样的功能，我们学习 C/C++ 也不能落后呀😄

<!--more-->

于是Google搜索了相关资料，在这里 [is-there-an-interpreter-for-c][is-there-an-interpreter-for-c] 找到了很多类似的工具，如 [cling][cling]、[Ch standard][Ch standard]、[TCC][TCC]、[picoc][picoc]、[Cint][Cint]、[iGCC][iGCC]、[ccons][ccons]、[CompCert][CompCert] 等，看了一下这些项目最近的活跃度，最活跃的是[cling][cling]。

从 [https://root.cern.ch/download/cling/][https://root.cern.ch/download/cling/] 可以下载到编译好的安装包，但只有Linux、MacOS版本的，因为有时候会用到Windows，所以就想找找看，但网上虽然有编译脚本、教程([cling-build-instructions][cling-build-instructions]、[The (Failed) Adventures in building Cling under Windows 10 with VS2017][The (Failed) Adventures in building Cling under Windows 10 with VS2017])，但没有提供下载的，所以就自己编译了一个。

我的编译过程参考[cling-build-instructions][cling-build-instructions]如下

```shell
git clone http://root.cern.ch/git/llvm.git src
cd src
git checkout cling-patches
cd tools
git clone http://root.cern.ch/git/cling.git
git clone http://root.cern.ch/git/clang.git
cd clang
git checkout cling-patches
# 这一步需要切换到src外层目录
# cd ../..
cd ../../..
mkdir build
cd build
# add parallel build support
# https://stackoverflow.com/questions/601970/how-do-i-utilise-all-the-cores-for-nmake
set CL=/MP
# 我在src、build同级目录新建了一个release目录用来存放编译好的文件，即安装文件位置
# cmake -DCMAKE_INSTALL_PREFIX=[Install Path] -DCMAKE_BUILD_TYPE=[Build configuration, e.g. Release or Debug] ..\src
cmake -DCMAKE_INSTALL_PREFIX=../release -DCMAKE_BUILD_TYPE=Release ..\src
cmake --build .
cmake --build . --target install
```


[is-there-an-interpreter-for-c]: https://stackoverflow.com/questions/584714/is-there-an-interpreter-for-c
[cling]: https://github.com/root-project/cling
[Ch standard]: http://www.softintegration.com/products/chstandard/
[TCC]: http://bellard.org/tcc/
[picoc]: https://github.com/zsaleeba/picoc
[Cint]: http://www.hanno.jp/gotom/Cint.html
[iGCC]: http://www.artificialworlds.net/wiki/IGCC/IGCC
[ccons]: https://github.com/asvitkine/ccons
[CompCert]: https://github.com/AbsInt/CompCert
[https://root.cern.ch/download/cling/]: https://root.cern.ch/download/cling/
[cling-build-instructions]: https://root.cern.ch/cling-build-instructions
[The (Failed) Adventures in building Cling under Windows 10 with VS2017]: https://github.com/root-project/cling/issues/186

