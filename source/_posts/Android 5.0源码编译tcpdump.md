---
title: Android 5.0源码编译tcpdump
tags:
  - android
  - tcpdump
id: 616
categories:
  - 学习
date: 2014-12-05 14:31:10
---

之前我写过一篇文章"[非Android源码中编译tcpdump](http://202.203.209.55:8080/?p=372)"，今天我终于使用AWS EC2作为代理服务器，完整的更新下载了Android源码（更新后，我的.repo居然有42G，checkout之后的纯代码有大概15G，Android源码也是越来越大了！一不小心免费的AWS的流量就超了7G多，$1.5就没了），所以就想在Android源码中编译tcpdump，体会原汁原味。以下记录我编译的一些过程。

<!--more-->

1. 首先整个源码编译一遍，我使用make -j8，大概两小时完成
2. cd到external/tcpdump，修改Android.mk，让编译出的tcpdump可以在Android 5.0上运行
```shell
liudonghua@liudonghua-Ubuntu:~/android$ cd external/tcpdump/
liudonghua@liudonghua-Ubuntu:~/android/external/tcpdump$ git diff
diff --git a/Android.mk b/Android.mk
index 348d8b0..8dc7a4e 100644
--- a/Android.mk
+++ b/Android.mk
@@ -38,6 +38,9 @@ LOCAL_SRC_FILES:=\

 LOCAL_CFLAGS := -O2 -g
 LOCAL_CFLAGS += -DHAVE_CONFIG_H -D_U_="__attribute__((unused))"
+LOCAL_CFLAGS += -pie -fPIE
+
+LOCAL_LDFLAGS := -pie -fPIE

 LOCAL_C_INCLUDES += \
        $(LOCAL_PATH)/missing\
liudonghua@liudonghua-Ubuntu:~/android/external/tcpdump$
```
3. 执行mm，后面的编译过程如下

```shell
liudonghua@liudonghua-Ubuntu:~/android/external/tcpdump$ mm
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=5.0.50.50.50.50
TARGET_PRODUCT=aosp_hammerhead
TARGET_BUILD_VARIANT=userdebug
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv7-a-neon
TARGET_CPU_VARIANT=krait
TARGET_2ND_ARCH=
TARGET_2ND_ARCH_VARIANT=
TARGET_2ND_CPU_VARIANT=
HOST_ARCH=x86_64
HOST_OS=linux
HOST_OS_EXTRA=Linux-3.13.0-39-generic-x86_64-with-Ubuntu-14.04-trusty
HOST_BUILD_TYPE=release
BUILD_ID=AOSP
OUT_DIR=out
============================================
PRODUCT_COPY_FILES device/generic/goldfish/data/etc/apns-conf.xml:system/etc/apns-conf.xml ignored.
No private recovery resources for TARGET_DEVICE hammerhead
make: Entering directory `/home/liudonghua/android'
make: *** No rule to make target `out/target/product/hammerhead/obj/STATIC_LIBRARIES/libpcap_intermediates/export_includes', needed by `out/target/product/hammerhead/obj/EXECUTABLES/tcpdump_intermediates/import_includes'.  Stop.
make: Leaving directory `/home/liudonghua/android'

#### make failed to build some targets (1 seconds) ####

liudonghua@liudonghua-Ubuntu:~/android/external/tcpdump$cd ../libpcap
/home/liudonghua/android/external/libpcap
liudonghua@liudonghua-Ubuntu:~/android/external/libpcap$ mm
.................................
Export includes file: external/libpcap/Android.mk -- out/target/product/hammerhead/obj/STATIC_LIBRARIES/libpcap_intermediates/export_includes
target StaticLib: libpcap (out/target/product/hammerhead/obj/STATIC_LIBRARIES/libpcap_intermediates/libpcap.a)
make: Leaving directory `/home/liudonghua/android'

#### make completed successfully (9 seconds) ####

liudonghua@liudonghua-Ubuntu:~/android/external/libpcap$
liudonghua@liudonghua-Ubuntu:~/android/external/libpcap$ cd -
/home/liudonghua/android/external/tcpdump
liudonghua@liudonghua-Ubuntu:~/android/external/tcpdump$ mm
.................................
target Executable: tcpdump (out/target/product/hammerhead/obj/EXECUTABLES/tcpdump_intermediates/LINKED/tcpdump)
target Symbolic: tcpdump (out/target/product/hammerhead/symbols/system/xbin/tcpdump)
Export includes file: external/tcpdump/Android.mk -- out/target/product/hammerhead/obj/EXECUTABLES/tcpdump_intermediates/export_includes
target Strip: tcpdump (out/target/product/hammerhead/obj/EXECUTABLES/tcpdump_intermediates/tcpdump)
Install: out/target/product/hammerhead/system/xbin/tcpdump
make: Leaving directory `/home/liudonghua/android'

#### make completed successfully (34 seconds) ####

liudonghua@liudonghua-Ubuntu:~/android/external/tcpdump$
liudonghua@liudonghua-Ubuntu:~/android$ file out/target/product/hammerhead/obj/EXECUTABLES/tcpdump_intermediates/tcpdump
out/target/product/hammerhead/obj/EXECUTABLES/tcpdump_intermediates/tcpdump: ELF 32-bit LSB  shared object, ARM, EABI5 version 1 (SYSV), dynamically linked (uses shared libs), stripped
```
如果之前没有源码编译过会mm libpcap时会有如下错误
```shell
liudonghua@liudonghua-Ubuntu:~/android/external/libpcap$ mm
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=5.0.50.50.50.50
TARGET_PRODUCT=aosp_arm
TARGET_BUILD_VARIANT=eng
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv7-a
TARGET_CPU_VARIANT=generic
TARGET_2ND_ARCH=
TARGET_2ND_ARCH_VARIANT=
TARGET_2ND_CPU_VARIANT=
HOST_ARCH=x86_64
HOST_OS=linux
HOST_OS_EXTRA=Linux-3.13.0-39-generic-x86_64-with-Ubuntu-14.04-trusty
HOST_BUILD_TYPE=release
BUILD_ID=AOSP
OUT_DIR=out
============================================
make: Entering directory `/home/liudonghua/android'
make: *** No rule to make target `out/target/product/generic/obj/SHARED_LIBRARIES/libc++_intermediates/export_includes', needed by `out/target/product/generic/obj/STATIC_LIBRARIES/libpcap_intermediates/import_includes'.  Stop.
make: Leaving directory `/home/liudonghua/android'

#### make failed to build some targets (1 seconds) ####

liudonghua@liudonghua-Ubuntu:~/android/external/libpcap$
```

其实仔细看m\mm\mmm的来源，我们完全可以用mma\mmma来代替解决编译依赖关系
```shell
liudonghua@liudonghua-Ubuntu:~/android$ head -n 20 build/envsetup.sh 
function hmm() {
cat <<EOF
Invoke ". build/envsetup.sh" from your shell to add the following functions to your environment:
- lunch:   lunch <product_name>-<build_variant>
- tapas:   tapas [<App1> <App2> ...] [arm|x86|mips|armv5|arm64|x86_64|mips64] [eng|userdebug|user]
- croot:   Changes directory to the top of the tree.
- m:       Makes from the top of the tree.
- mm:      Builds all of the modules in the current directory, but not their dependencies.
- mmm:     Builds all of the modules in the supplied directories, but not their dependencies.
           To limit the modules being built use the syntax: mmm dir/:target1,target2.
- mma:     Builds all of the modules in the current directory, and their dependencies.
- mmma:    Builds all of the modules in the supplied directories, and their dependencies.
- cgrep:   Greps on all local C/C++ files.
- ggrep:   Greps on all local Gradle files.
- jgrep:   Greps on all local Java files.
- resgrep: Greps on all local res/*.xml files.
- sgrep:   Greps on all local source files.
- godir:   Go to the directory containing a file.

Look at the source to view more functions. The complete list is:
liudonghua@liudonghua-Ubuntu:~/android$
```