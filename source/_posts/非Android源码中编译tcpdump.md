---
title: 非Android源码中编译tcpdump
tags:
  - android
  - ndk
  - tcpdump
id: 372
categories:
  - 学习
date: 2014-10-27 16:44:35
---

之前一篇[文章](http://202.203.209.55:8080/?p=367)提到的[以前下载的tcpdump](http://www.strazzere.com/android/tcpdump)在Android 5.0上运行出错，并且给出了解决办法，即编译时添加"-fPIE -pie"参数，简单的做法就是在Android源码中的external/tcpdump/Android.mk中添加这个参数，但如果没有下载Android源码，使用NDK可以编译出来吗？

<!--more-->

我测试了一下，因为Android源码里的tcpdump的Android.mk涉及到很多其他库的头文件、静态库、动态库等，如下所示的是tcpdump中Android.mk中的部分内容

```shell
LOCAL_C_INCLUDES += \
$(LOCAL_PATH)/missing\
external/openssl/include\
external/libpcap

LOCAL_SHARED_LIBRARIES += libssl libcrypto

LOCAL_STATIC_LIBRARIES += libpcap
```

而且libpcap、openssl也涉及到其他库，即有多重依赖关系，直接使用ndk-build非常困难，后来查阅资料在[这里](http://omappedia.org/wiki/USB_Sniffing_with_tcpdump)找到一些有用信息——直接使用ndk-build内部的交叉编译器(arm-linux-androideabi-*)编译，按照文章里提到的方式去编译，大致命令如下（在Windows平台上交叉编译很不方便，以下是在Linux下操作的）

```shell
mkdir ~/tcpdump
cd ~/tcpdump
wget http://www.tcpdump.org/release/libpcap-1.6.2.tar.gz
wget http://www.tcpdump.org/release/tcpdump-4.6.2.tar.gz
tar -xzvf libpcap-1.6.2.tar.gz
tar -xzvf tcpdump-4.6.2.tar.gz
mkdir toolchain
%NDK_HOME%/build/tools/make-standalone-toolchain.sh --platform=android-21 --install-dir=~/tcpdump/toolchain
export PATH=~/tcpdump/toolchain/bin:$PATH
export CC=arm-linux-androideabi-gcc
export RANLIB=arm-linux-androideabi-ranlib
export AR=arm-linux-androideabi-ar
export LD=arm-linux-androideabi-ld
cd libpcap-1.6.2
./configure --host=arm-linux --with-pcap=~/tcpdump/libpcap-1.6.2 ac_cv_linux_vers=2
make
cd ../tcpdump-4.6.2
sed -i".bak" "s/setprotoent/\/\/setprotoent/g" print-isakmp.c
sed -i".bak" "s/endprotoent/\/\/endprotoent/g" print-isakmp.c
./configure --host=arm-linux --with-pcap=linux --with-crypto=no ac_cv_linux_vers=2 --disable-ipv6
vi Makefile # 在CFLAGS、LDFLAGS中添加"-fPIE -pie"
make # 或者不用做上面那一步，使用make CFLAGS="-DNBBY=8 -fPIE -pie"
# 最后编译完之后在根目录下即有tcpdump
file tcpdump
tcpdump: ELF 32-bit LSB  shared object, ARM, EABI5 version 1 (SYSV), dynamically linked (uses shared libs), not stripped
```

其中在tcpdump-4.6.2中configure时需要添加"--disable-ipv6"，否则会提示

```shell
checking whether to enable ipv6... yes
checking ipv6 stack type... linux-libinet6
You do not have inet6 library, using libc
checking for library containing getaddrinfo... none required
checking getaddrinfo bug... buggy
Fatal: You must get working getaddrinfo() function.
       or you can specify "--disable-ipv6".
```

但即使添加了也会有如下警告，但不影响后面的编译及tcpdump的正常使用

```shell
config.status: executing default-1 commands
configure: WARNING: unrecognized options: --with-pcap
```

接着就可以推送到手机中使用了

```shell
adb push tcpdump /sdcard/
adb shell
su
mount -o remount,rw /system
mv /sdcard/tcpdump /system/xbin/
chmod 4755 /system/xbin/tcpdump
```

但这样非root用户运行tcpdump还是会报如下错误，所以后面还是得su之后抓包

```shell
tcpdump: Can't open netlink socket 13:Permission denied
```

这里是我编译的[tcpdump](/resources/2014/10/tcpdump.zip)，方便别人直接下载使用