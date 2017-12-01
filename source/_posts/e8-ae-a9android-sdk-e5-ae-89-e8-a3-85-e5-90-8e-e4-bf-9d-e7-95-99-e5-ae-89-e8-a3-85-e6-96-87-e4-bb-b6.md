---
title: 让Android SDK安装后保留安装文件
tags:
  - android
  - android sdk
id: 738
categories:
  - 学习
date: 2015-04-26 20:04:42
---

在国内安装Android SDK里的内容是非常麻烦的，Google服务器几乎不可用，必须要用代理才可以。之前还遇到过用代理也不可用的情况下，直接通过某种方式（如浏览器）下载回来安装的zip文件，然后放到SDK的temp目录里，这样安装时检测到已有下载好的文件就不会再联网下载。<!--more-->

所以我就想，如果在一个没有安装内容的空的SDK里，完全安装时不删除安装文件，那么把一个空的SDK+temp下的安装文件拷贝到其他电脑上安装就会非常快，我现在SDK已经有30+G，零碎下文件非常多，拷贝共享非常慢，即使压缩解压缩也要很长世间，如果这种方式实现的话，那么共享SDK就会很快很方便了。

所以自然而然的想到找到SDK中下载、安装、删除安装文件这里的代码，然后进行修改。
按照这个思路，首先找到SDK的源码（在git库"https://android.googlesource.com/platform/tools/swt"下的sdkmanager中），然后沿着"sdkmanager\app\src\main\java\com\android\sdkmanager\Main.java"找到在主界面中点击安装后弹出的接受许可窗口SdkUpdaterChooserDialog，接着找到SwtUpdaterData，在里面发现主要线索！

```java
    public List<Archive> updateOrInstallAll_WithGUI(
            Collection<Archive> selectedArchives,
            boolean includeObsoletes,
            int flags) {
        // ........
        ArrayList<ArchiveInfo> result = dialog.getResult();
        if (result != null && result.size() > 0) {
            return installArchives(result, flags);
        }
        return null;
    }
```

但线索在这里断了，因为在这个git库中所有代码中都搜索过，没有installArchives，后来想到可能是其父类里面的方法，所以继续找UpdaterData，终于在另一个git库"https://android.googlesource.com/platform/tools/base"下的sdklib中找到了，并且继续往下找，...(省略一些过程)，最终终于定位到[ArchiveInstaller](https://android.googlesource.com/platform/tools/base/+/master/sdklib/src/main/java/com/android/sdklib/internal/repository/archives/ArchiveInstaller.java#114)中的install方法

```java
    public boolean install(ArchiveReplacement archiveInfo,
            String osSdkRoot,
            boolean forceHttp,
            SdkManager sdkManager,
            DownloadCache cache,
            ITaskMonitor monitor) {
        Archive newArchive = archiveInfo.getNewArchive();
        Package pkg = newArchive.getParentPackage();
        String name = pkg.getShortDescription();
        if (newArchive.isLocal()) {
            // This should never happen.
            monitor.log("Skipping already installed archive: %1$s for %2$s",
                    name,
                    newArchive.getOsDescription());
            return false;
        }
        // In detail mode, give us a way to force install of incompatible archives.
        boolean checkIsCompatible = System.getenv(ENV_VAR_IGNORE_COMPAT) == null;
        if (checkIsCompatible && !newArchive.isCompatible()) {
            monitor.log("Skipping incompatible archive: %1$s for %2$s",
                    name,
                    newArchive.getOsDescription());
            return false;
        }
        Pair<File, File> files = downloadFile(newArchive, osSdkRoot, cache, monitor, forceHttp);
        File tmpFile   = files == null ? null : files.getFirst();
        File propsFile = files == null ? null : files.getSecond();
        if (tmpFile != null) {
            // Unarchive calls the pre/postInstallHook methods.
            if (unarchive(archiveInfo, osSdkRoot, tmpFile, sdkManager, monitor)) {
                monitor.log("Installed %1$s", name);
                // Delete the temp archive if it exists, only on success
                mFileOp.deleteFileOrFolder(tmpFile);
                mFileOp.deleteFileOrFolder(propsFile);
                return true;
            }
        }
        return false;
    }
```

**只要理论上只要把以上高亮的第33，34行注释掉或删掉就可以了。**

我首先想到的是既然已经有这个Java源文件，那能不能直接不管它所依赖的其他类库强制编译，但这种方法经验证是不行的（可参考[compile-java-class-with-missing-code-parts](http://stackoverflow.com/questions/4661951/compile-java-class-with-missing-code-parts)、[disabling-compile-time-dependency-checking-when-compiling-java-classes](http://stackoverflow.com/questions/1537714/disabling-compile-time-dependency-checking-when-compiling-java-classes)，经过后面的错误大概可以知道，class文件在编译时需要寻找所依赖的那些类/接口的真实的方法签名，即有些信息是需要编译时静态写入class文件中，可能有些类似于"静态绑定"）。后来想到之前用过的一种方法（Mock类方法）——在eclipse中新建一个Java工程，加入ArchiveInstaller类（注意包名不能变），根据需要修改，然后再添加这个类所以来的其他类及方法（只需要空类、空方法，总之是空的，总之就是让这个工程没有至少语法错误，能编译通过），没想到这个工程一片红，只能一个一个改，遇到新的类、方法就添加，半个多小时过去了，终于修改至这个工程没有报错了（注意有些地方不能用class，要用interface、annotation以及还有一些throws XXXException、extends XXX、implements XXX）。紧接着替换SDK目录下tools\lib\sdklib.jar中的ArchiveInstaller.class，验证发现有如下错误

[text]
Preparing to install archives
Unexpected Error installing 'Intel x86 Emulator Accelerator (HAXM installer), revision 5.3': java.lang.IncompatibleClassChangeError: Found interface com.android.sdklib.internal.repository.ITaskMonitor, but class was expected
Done. Nothing was installed.
[/text]

即没注意到ITaskMonitor是接口而不是类，修改后又提示

[text]
Preparing to install archives
Unexpected Error installing 'Intel x86 Emulator Accelerator (HAXM installer), revision 5.3': java.lang.NoSuchMethodError: com.android.sdklib.internal.repository.ITaskMonitor.setDescription(Ljava/lang/String;Ljava/lang/String;)V
Done. Nothing was installed.
[/text]

因为我根据eclipse代码提示自动生成的ITaskMonitor.setDescription方法原型是setDescription(Ljava/lang/String;Ljava/lang/String;)V，而实际上真实的是setDescription(Ljava/lang/String;Ljava/lang/Object;)V
继续修改后又提示

[text]
Preparing to install archives
Downloading Intel x86 Emulator Accelerator (HAXM installer), revision 5.3
Unexpected Error installing 'Intel x86 Emulator Accelerator (HAXM installer), revision 5.3': java.lang.NoSuchMethodError: com.android.sdklib.internal.repository.ITaskMonitor.log(Ljava/lang/String;Ljava/lang/String;)V
Done. Nothing was installed.
[/text]

彻底无语了，因为ArchiveInstaller中有多个地方调用ITaskMonitor.log，有不同的参数格式，我就生成了多个重载的ITaskMonitor.log的方法，没想到其真实的实现中直接是void log(String format, Object...args)，如果再这样一个一个慢慢修该，可能几天都改不好吧！
所以这种方法只能放弃(这种方法只适用于代码结构简单的情况)

另一种方法就是看看有没有直接编辑class文件的工具，找到了两个jbe和ce，jbe要稍微好用一点点，看一下原始的和上一部修改过的ArchiveInstaller的installer最后部分的区别，如下图所示
[![jbe_ArchiveInstaller](/resources/2015/04/jbe_ArchiveInstaller.png)](/resources/2015/04/jbe_ArchiveInstaller.png)
删除选中的那块（应该是对应着"mFileOp.deleteFileOrFolder(tmpFile);mFileOp.deleteFileOrFolder(propsFile);"这两行代码，但保存时出错，应该是删除之后破坏了class文件结构），所以这种方法需要对class文件结构（以及汇编指令？）有深入了解的用户才能修改，这里面涉及到Fields/Methods/Constant Pool/Interfaces/Attributes等内容，不是简简单单的和代码一样很直接的逻辑，后面有时间找相关资料深入学习一下才行！！！

最后我突然想到如果把SDK目录tools\lib中的所有jar包（应该是包含了需要用到的所以类库）都作为这一个工程的依赖类库，或许可以不用那些Mocks，尝试了一下居然可以，工程虽然报错了，但还是编译出修改过的ArchiveInstaller，然后替换sdklib.jar，测试一下，果然下载安装以后不会删除安装文件了！！！

其实如果你在Android源码下工作的话，修改后，直接make sdk就可以了，不需要这么麻烦~~~![](http://202.203.209.55:8080/wp-content/plugins/wp-emoji-one/icons/1F631.png)

附上修改过的[ArchiveInstaller](/resources/2015/04/ArchiveInstaller.class)