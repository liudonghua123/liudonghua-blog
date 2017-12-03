---
title: Lua Windows下VS编译
tags:
  - lua
  - vs
id: 607
categories:
  - 学习
date: 2014-12-04 15:17:28
---

Lua是一门短小精炼，可扩展性很强的脚本语言，相比Python、Ruby等其他脚本语言，虽然本身核心库提供的功能有限，但运行效率据说是最好的，在一些游戏引擎中被使用，以被人了解，很多人提到lua就会联想到游戏引擎，其实lua还有很多非常广泛的用途，主要用在一些嵌入式、对性能要求很高、资源很有限（KB级内存）的领域。

<!--more-->

Linux、Mac等类Unix下编译安装lua都非常方便，wget源码之后，configure、make、make install即可搞定，但Windows下就不是很方便，虽然可以在Cgywin、MinGW等环境中使用Windows版的gcc、g++编译，不但麻烦，而且使用luarocks安装第三方需要编译的库时由于使用VS的cl、link等编译、链接会导致无法正确链接之前用其他编译器、链接器生成的dll或lib。所以最好的方式还是使用Windows原生的编译器编译，其过程步骤如下（我使用的是VS2013，其他版本大致类似）

1.新建一个Win32控制台项目，如lua52（默认也会创建一个lua的解决方案，解决方案下可包括多个项目），Application Settings中选择DLL及选中Empty project

2.添加源码src下的*.h,*.hpp头文件到Header Files，添加*.c（除lua.c\luac.c）到Source Files

3.最重要的一步，项目属性中Configuration Properties - C/C++ - Command Line的Additional Options中添加**/DLUA_BUILD_AS_DLL** /D _CRT_SECURE_NO_WARNINGS，这里的/DLUA_BUILD_AS_DLL是必须的（因为在VS里编译动态库时必须要用__declspec(dllexport)、__declspec(dllimport)，而lua.h中所有的待导出函数都有LUA_API，再查看其宏定义位置就可找到luaconf.h中的声明，见下图），如果没有，虽然可以编译出DLL，但没有对应的LIB，而其他可执行项目链接时不管是动态链接、静态链接都是链接到LIB的，区别是如果是动态链接，链接到LIB后再由LIB链接DLL，静态链接就直接链接LIB结束；/D _CRT_SECURE_NO_WARNINGS是避免很多控制台编译警告。
[![lua_DLUA_BUILD_AS_DLL](/resources/2014/12/lua_DLUA_BUILD_AS_DLL.png)](/resources/2014/12/lua_DLUA_BUILD_AS_DLL.png)
[![lua_project_compile_command_line](/resources/2014/12/lua_project_compile_command_line.png)](/resources/2014/12/lua_project_compile_command_line.png)
4.然后再在这个解决方案下新建Win32控制台项目，如lua，Application Settings中选择Console Application及选中Empty project

5.在lua项目属性中的设置项目依赖，即可执行项目lua依赖动态库项目lua52，如下图所示，然后在Configuration Properties - C/C++ - General的Additional Include Directories中添加源码src路径
[![lua_project_dependence_references_settings](/resources/2014/12/lua_project_dependence_references_settings.png)](/resources/2014/12/lua_project_dependence_references_settings.png)

6.添加lua.c到Source Files中
这样编译解决方案时就可以编译出DLL、LIB、EXE等文件了

7.luac和lua操作类似

注：动态库编译出的lib是[import libraries](http://en.wikipedia.org/wiki/Dynamic-link_library#Import_libraries)，静态库编译出的lib才是static/share libraries
> ### <span id="Import_libraries" class="mw-headline">Import libraries</span>
> 
> Like static libraries, import libraries for DLLs are noted by the .lib file extension. For example, [kernel32.dll](http://en.wikipedia.org/wiki/Kernel32.dll "Kernel32.dll"), the primary dynamic library for Windows' base functions such as file creation and memory management, is linked via kernel32.lib.
> 
> 
> Linking to dynamic libraries is usually handled by linking to an import library when building or linking to create an executable file. The created executable then contains an import address table (IAT) by which all DLL function calls are referenced (each referenced DLL function contains its own entry in the IAT). At run-time, the IAT is filled with appropriate addresses that point directly to a function in the separately loaded DLL.
其实lua代码写的很规范，程序也不是很大，可以深入分析学习以提高自己的C/C++编程能力，源码目录中只包括GNU makefile，不能使用VS的nmake，不过写一个跨平台的cmake应该不难。

参考资料
1.[Walkthrough: Creating and Using a Dynamic Link Library (C++)](http://msdn.microsoft.com/en-us/library/ms235636.aspx)
2.[Microsoft Visual C++ Static and Dynamic Libraries](http://www.codeproject.com/Articles/85391/Microsoft-Visual-C-Static-and-Dynamic-Libraries)
3.[Walkthrough: Compiling a Native C++ Program on the Command Line](http://msdn.microsoft.com/en-us/library/ms235639.aspx)
4.[Windows下Visual Studio 2013编译Lua 5.2.3](http://www.cnblogs.com/junchu25/p/3626280.html)
5.[windows下编译lua 5.2.x](http://z1y1m1.blog.163.com/blog/static/518373272014322102138399/)
6.[LuaBinaries](http://sourceforge.net/projects/luabinaries/)
7.[dllexport, dllimport](http://msdn.microsoft.com/en-us/library/3y1sfaz2.aspx)