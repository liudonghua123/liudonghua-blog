---
title: 一次失败的替换Android系统文件
tags:
  - android
  - hierarchyviewer
id: 628
categories:
  - 学习
date: 2014-12-05 17:20:49
---

在Android 5.0之前，root后经过[一些操作](http://blog.apkudo.com/2012/07/26/enabling-hierarchyviewer-on-rooted-android-devices/)就可以在真机上使用hierarchyviewer查看应用界面布局结构，非常实用的一项功能，但我用之前的那些步骤在新系统上操作时，到这里卡住了

[shell]
E:\hierarchyviewer_enable&gt;java -jar D:\android\decompile\baksmali.jar -x -a 21 -c /system/framework/core-libart.jar:/system/framework/conscrypt.jar:/system/framework/okhttp.jar:/system/framework/core-junit.jar:/system/framework/bouncycastle.jar:/system/framework/ext.jar:/system/framework/framework.jar:/system/framework/telephony-common.jar:/system/framework/voip-common.jar:/system/framework/ims-common.jar:/system/framework/mms-common.jar:/system/framework/android.policy.jar:/system/framework/apache-xml.jar  ./system/framework/services.odex
Exception in thread &quot;main&quot; org.jf.util.ExceptionWithContext: .\system\framework\services.odex is not an apk, dex file or odex file.
        at org.jf.dexlib2.DexFileFactory.loadDexFile(DexFileFactory.java:111)
        at org.jf.dexlib2.DexFileFactory.loadDexFile(DexFileFactory.java:54)
        at org.jf.baksmali.main.main(main.java:252)
[/shell]

大概是因为Android 5.0变化了很多，使用的ART代替了Dalvik，并且/system/framework下的services.jar/services.odex位置也变了，原来services.odex在/system/framework下，现在位于/system/framework/arm，或者可能是smali/baksmali不支持新的odex格式，还有待继续研究。

后来我就想在Android源码里编译，然后替换手机里的services.jar/odex，使用和我手机匹配的combo（lunch之后选择"13\. aosp_hammerhead-userdebug"），初始编译成功，接着我修改WindowManagerService.java之后再增量编译。

[shell]
liudonghua@liudonghua-Ubuntu:~/android/frameworks/base$ git diff
diff --git a/services/core/java/com/android/server/wm/WindowManagerService.java b/services/core/java/com/android/server/wm/WindowManagerService.java
index 1c03ced..6aad481 100644
--- a/services/core/java/com/android/server/wm/WindowManagerService.java
+++ b/services/core/java/com/android/server/wm/WindowManagerService.java
@@ -6648,8 +6648,9 @@ public class WindowManagerService extends IWindowManager.Stub
     }

     private boolean isSystemSecure() {
-        return &quot;1&quot;.equals(SystemProperties.get(SYSTEM_SECURE, &quot;1&quot;)) &amp;&amp;
-                &quot;0&quot;.equals(SystemProperties.get(SYSTEM_DEBUGGABLE, &quot;0&quot;));
+        //return &quot;1&quot;.equals(SystemProperties.get(SYSTEM_SECURE, &quot;1&quot;)) &amp;&amp;
+        //        &quot;0&quot;.equals(SystemProperties.get(SYSTEM_DEBUGGABLE, &quot;0&quot;));
+        return false;
     }

     /**
liudonghua@liudonghua-Ubuntu:~/android/frameworks/base$
[/shell]

使用mma编译

[shell highlight="79,103"]
liudonghua@liudonghua-Ubuntu:~/android/frameworks/base$ mm WITH_DEXPREOPT=true
...................
target Dex: framework
Copying: out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/classes.dex
target Jar: framework (out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/javalib.jar)
Install: out/target/product/hammerhead/system/framework/framework.jar
make: *** No rule to make target `out/host/linux-x86/bin/dex2oatd', needed by `out/target/product/hammerhead/dex_bootjars/system/framework/arm/boot.art'.  Stop.
make: Leaving directory `/home/liudonghua/android'

#### make failed to build some targets (53 seconds) ####

liudonghua@liudonghua-Ubuntu:~/android/frameworks/base$
liudonghua@liudonghua-Ubuntu:~/android/frameworks/base$ ll ~/android/out/host/linux-x86/bin/dex2oatd
-rwxrwxr-x 1 liudonghua liudonghua 3883976 Dec  5 14:40 /home/liudonghua/android/out/host/linux-x86/bin/dex2oatd*
liudonghua@liudonghua-Ubuntu:~/android/frameworks/base$ ll ~/android/out/target/product/hammerhead/dex_bootjars/system/framework/arm/boot.art
-rw-r--r-- 1 liudonghua liudonghua 10338304 Dec  5 14:40 /home/liudonghua/android/out/target/product/hammerhead/dex_bootjars/system/framework/arm/boot.art
liudonghua@liudonghua-Ubuntu:~/android/frameworks/base$
liudonghua@liudonghua-Ubuntu:~/android/frameworks/base$ mma WITH_DEXPREOPT=true
...................
target Java: services.accessibility (out/target/common/obj/JAVA_LIBRARIES/services.accessibility_intermediates/classes)
Note: frameworks/base/services/accessibility/java/com/android/server/accessibility/AccessibilityInputFilter.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
Copying: out/target/common/obj/JAVA_LIBRARIES/services.accessibility_intermediates/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.accessibility_intermediates/emma_out/lib/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.accessibility_intermediates/classes.jar
target Static Jar: services.accessibility (out/target/common/obj/JAVA_LIBRARIES/services.accessibility_intermediates/javalib.jar)
target Java: services.appwidget (out/target/common/obj/JAVA_LIBRARIES/services.appwidget_intermediates/classes)
Copying: out/target/common/obj/JAVA_LIBRARIES/services.appwidget_intermediates/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.appwidget_intermediates/emma_out/lib/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.appwidget_intermediates/classes.jar
target Static Jar: services.appwidget (out/target/common/obj/JAVA_LIBRARIES/services.appwidget_intermediates/javalib.jar)
target Java: services.backup (out/target/common/obj/JAVA_LIBRARIES/services.backup_intermediates/classes)
Note: frameworks/base/services/backup/java/com/android/server/backup/BackupManagerService.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
Note: frameworks/base/services/backup/java/com/android/server/backup/BackupManagerService.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
Copying: out/target/common/obj/JAVA_LIBRARIES/services.backup_intermediates/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.backup_intermediates/emma_out/lib/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.backup_intermediates/classes.jar
target Static Jar: services.backup (out/target/common/obj/JAVA_LIBRARIES/services.backup_intermediates/javalib.jar)
target Java: services.print (out/target/common/obj/JAVA_LIBRARIES/services.print_intermediates/classes)
Copying: out/target/common/obj/JAVA_LIBRARIES/services.print_intermediates/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.print_intermediates/emma_out/lib/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.print_intermediates/classes.jar
target Static Jar: services.print (out/target/common/obj/JAVA_LIBRARIES/services.print_intermediates/javalib.jar)
target Java: services.restrictions (out/target/common/obj/JAVA_LIBRARIES/services.restrictions_intermediates/classes)
Copying: out/target/common/obj/JAVA_LIBRARIES/services.restrictions_intermediates/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.restrictions_intermediates/emma_out/lib/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.restrictions_intermediates/classes.jar
target Static Jar: services.restrictions (out/target/common/obj/JAVA_LIBRARIES/services.restrictions_intermediates/javalib.jar)
target Java: services.usage (out/target/common/obj/JAVA_LIBRARIES/services.usage_intermediates/classes)
Note: frameworks/base/services/usage/java/com/android/server/usage/UsageStatsDatabase.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
Copying: out/target/common/obj/JAVA_LIBRARIES/services.usage_intermediates/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.usage_intermediates/emma_out/lib/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.usage_intermediates/classes.jar
target Static Jar: services.usage (out/target/common/obj/JAVA_LIBRARIES/services.usage_intermediates/javalib.jar)
target Java: services.usb (out/target/common/obj/JAVA_LIBRARIES/services.usb_intermediates/classes)
Note: Some input files use or override a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
Copying: out/target/common/obj/JAVA_LIBRARIES/services.usb_intermediates/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.usb_intermediates/emma_out/lib/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.usb_intermediates/classes.jar
target Static Jar: services.usb (out/target/common/obj/JAVA_LIBRARIES/services.usb_intermediates/javalib.jar)
target Java: services.voiceinteraction (out/target/common/obj/JAVA_LIBRARIES/services.voiceinteraction_intermediates/classes)
Copying: out/target/common/obj/JAVA_LIBRARIES/services.voiceinteraction_intermediates/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.voiceinteraction_intermediates/emma_out/lib/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services.voiceinteraction_intermediates/classes.jar
target Static Jar: services.voiceinteraction (out/target/common/obj/JAVA_LIBRARIES/services.voiceinteraction_intermediates/javalib.jar)
target Java: services (out/target/common/obj/JAVA_LIBRARIES/services_intermediates/classes)
Copying: out/target/common/obj/JAVA_LIBRARIES/services_intermediates/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services_intermediates/emma_out/lib/classes-jarjar.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services_intermediates/classes.jar
Copying: out/target/common/obj/JAVA_LIBRARIES/services_intermediates/noproguard.classes.jar
target Dex: services
Copying: out/target/common/obj/JAVA_LIBRARIES/services_intermediates/classes.dex
target Jar: services (out/target/common/obj/JAVA_LIBRARIES/services_intermediates/javalib.jar)
Install: out/target/product/hammerhead/system/framework/services.jar
Dexpreopt Jar: services (out/target/product/hammerhead/obj/JAVA_LIBRARIES/services_intermediates/arm/javalib.odex)
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1786] out/host/linux-x86/bin/dex2oatd --runtime-arg -Xms64m --runtime-arg -Xmx512m --boot-image=out/target/product/hammerhead/dex_bootjars/system/framework/boot.art --dex-file=out/target/common/obj/JAVA_LIBRARIES/services_intermediates/javalib.jar --dex-location=/system/framework/services.jar --oat-file=out/target/product/hammerhead/obj/JAVA_LIBRARIES/services_intermediates/arm/javalib.odex --android-root=out/target/product/hammerhead/system --instruction-set=arm --instruction-set-variant=krait --instruction-set-features=default --include-patch-information --runtime-arg -Xnorelocate --no-include-debug-symbols
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406] compiler [Exclusive time] [Total time]
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]   0.121s dex2oat Setup
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]   0.002s/1.493s dex2oat Compile
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]     0.092s Resolve MethodsAndFields
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]     0.349s Verify Dex File
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]     0.002s InitializeNoClinit
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]     1.046s Compile Dex File
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]   0s/0.095s dex2oat Oat
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]     0s/0.028s dex2oat OatWriter
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]       0s Loading image checksum
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]       0s InitOatHeader
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]       0s InitOatDexFiles
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]       0s InitDexFiles
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]       0.004s InitOatClasses
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]       0.012s InitOatMaps
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]       0s InitOatCode
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]       0.011s InitOatCodeDexFiles
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]     0.066s dex2oat Write ELF
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]   0.166s dex2oat Flush ELF
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406] compiler: end, 1.877s
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1406]
dex2oatd I 28753 28753 art/dex2oat/dex2oat.cc:1595] dex2oat took 1.887s (threads: 4) arena alloc=9MB java alloc=4MB native alloc=17MB free=3MB
Install: out/target/product/hammerhead/system/framework/arm/services.odex
............
target Dex: wm
Copying: out/target/common/obj/JAVA_LIBRARIES/wm_intermediates/classes.dex
target Jar: wm (out/target/common/obj/JAVA_LIBRARIES/wm_intermediates/javalib.jar)
Install: out/target/product/hammerhead/system/framework/wm.jar
Dexpreopt Jar: wm (out/target/product/hammerhead/obj/JAVA_LIBRARIES/wm_intermediates/arm/javalib.odex)
dex2oatd I 29202 29202 art/dex2oat/dex2oat.cc:1786] out/host/linux-x86/bin/dex2oatd --runtime-arg -Xms64m --runtime-arg -Xmx512m --boot-image=out/target/product/hammerhead/dex_bootjars/system/framework/boot.art --dex-file=out/target/common/obj/JAVA_LIBRARIES/wm_intermediates/javalib.jar --dex-location=/system/framework/wm.jar --oat-file=out/target/product/hammerhead/obj/JAVA_LIBRARIES/wm_intermediates/arm/javalib.odex --android-root=out/target/product/hammerhead/system --instruction-set=arm --instruction-set-variant=krait --instruction-set-features=default --include-patch-information --runtime-arg -Xnorelocate --no-include-debug-symbols
dex2oatd I 29202 29202 art/dex2oat/dex2oat.cc:1595] dex2oat took 85.975ms (threads: 4) arena alloc=36KB java alloc=3KB native alloc=127KB free=1436KB
Install: out/target/product/hammerhead/system/framework/arm/wm.odex
make: Leaving directory `/home/liudonghua/android'

#### make completed successfully (51:09 (mm:ss)) ####

liudonghua@liudonghua-Ubuntu:~/android/frameworks/base$
[/shell]

之后把services.jar/services.odex取出来替换系统原来的文件（注意修复权限），但这次不像之前系统一样会自动给你重启然后重建dalvik-cache，我手动重启了也效果，也确认编译出的odex包含修改的代码（编译生成的中间文件在out/target/common/obj/JAVA_LIBRARIES/services_intermediates），后来尝试删除/data/dalvik-cache/arm/system@framework@services.jar@classes.dex想让他重建这个文件，但重启之后，过了很长时间始终在加载界面不动，最后就没办法，查看日志相关错误显示如下

[shell]
I/dex2oat ( 8445): /system/bin/dex2oat --zip-fd=6 --zip-location=/system/framework/services.jar --oat-fd=7 --oat-location=/data/dalvik-cache/arm/system@framework@services.jar@classes.dex --instruction-set=arm --instruction-set-features=div --runtime-arg -Xms64m --runtime-arg -Xmx512m
E/dex2oat ( 8445): Failed to open dex from file descriptor for zip file '/system/framework/services.jar': Entry not found
I/dex2oat ( 8445): dex2oat took 248.448ms (threads: 4)
E/installd(  202): DexInv: --- END '/system/framework/services.jar' --- status=0x0100, process failed
I/InstallerConnection( 8430): disconnecting...
E/installd(  202): eof
E/installd(  202): failed to read size
I/installd(  202): closing connection
E/Zygote  ( 8430): Zygote died with exception
E/Zygote  ( 8430): java.lang.RuntimeException: Missing class when invoking static main com.android.server.SystemServer
E/Zygote  ( 8430):      at com.android.internal.os.RuntimeInit.invokeStaticMain(RuntimeInit.java:204)
E/Zygote  ( 8430):      at com.android.internal.os.RuntimeInit.applicationInit(RuntimeInit.java:321)
E/Zygote  ( 8430):      at com.android.internal.os.RuntimeInit.zygoteInit(RuntimeInit.java:276)
E/Zygote  ( 8430):      at com.android.internal.os.ZygoteInit.handleSystemServerProcess(ZygoteInit.java:537)
E/Zygote  ( 8430):      at com.android.internal.os.ZygoteInit.startSystemServer(ZygoteInit.java:624)
E/Zygote  ( 8430):      at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:686)
E/Zygote  ( 8430): Caused by: java.lang.ClassNotFoundException: com.android.server.SystemServer
E/Zygote  ( 8430):      at java.lang.Class.classForName(Native Method)
E/Zygote  ( 8430):      at java.lang.Class.forName(Class.java:308)
E/Zygote  ( 8430):      at com.android.internal.os.RuntimeInit.invokeStaticMain(RuntimeInit.java:202)
E/Zygote  ( 8430):      ... 5 more
E/Zygote  ( 8430): Caused by: java.lang.ClassNotFoundException: Didn't find class &quot;com.android.server.SystemServer&quot; on path: DexPathList[[zip file &quot;/system/framework/services.jar&quot;, zip file &quot;/system/framework/ethernet-service.jar&quot;, zip file &quot;/system/framework/wifi-service.jar&quot;],nativeLibraryDirectories=[/vendor/lib, /system/lib]]
E/Zygote  ( 8430):      at dalvik.system.BaseDexClassLoader.findClass(BaseDexClassLoader.java:56)
E/Zygote  ( 8430):      at java.lang.ClassLoader.loadClass(ClassLoader.java:511)
E/Zygote  ( 8430):      at java.lang.ClassLoader.loadClass(ClassLoader.java:469)
E/Zygote  ( 8430):      ... 8 more
E/Zygote  ( 8430):      Suppressed: java.io.IOException: Zip archive '/system/framework/services.jar' doesn't contain classes.dex (error msg: Entry not found)
E/Zygote  ( 8430):              at dalvik.system.DexFile.openDexFileNative(Native Method)
E/Zygote  ( 8430):              at dalvik.system.DexFile.openDexFile(DexFile.java:295)
E/Zygote  ( 8430):              at dalvik.system.DexFile.&lt;init&gt;(DexFile.java:80)
E/Zygote  ( 8430):              at dalvik.system.DexFile.&lt;init&gt;(DexFile.java:59)
E/Zygote  ( 8430):              at dalvik.system.DexPathList.loadDexFile(DexPathList.java:262)
E/Zygote  ( 8430):              at dalvik.system.DexPathList.makeDexElements(DexPathList.java:231)
E/Zygote  ( 8430):              at dalvik.system.DexPathList.&lt;init&gt;(DexPathList.java:109)
E/Zygote  ( 8430):              at dalvik.system.BaseDexClassLoader.&lt;init&gt;(BaseDexClassLoader.java:48)
E/Zygote  ( 8430):              at dalvik.system.PathClassLoader.&lt;init&gt;(PathClassLoader.java:38)
E/Zygote  ( 8430):              at com.android.internal.os.ZygoteInit.handleSystemServerProcess(ZygoteInit.java:530)
E/Zygote  ( 8430):              ... 2 more
E/Zygote  ( 8430):      Caused by: java.io.IOException: Failed to open oat file from dex location '/system/framework/services.jar'
E/Zygote  ( 8430):              ... 12 more
E/Zygote  ( 8430):      Caused by: java.io.IOException: Failed to open oat file from /system/framework/arm/services.odex (error Invalid oat magic for '/system/framework/arm/services.odex') (no dalvik_cache availible) and relocation failed.
E/Zygote  ( 8430):              ... 12 more
E/Zygote  ( 8430):      Caused by: java.io.IOException:
E/Zygote  ( 8430):              ... 12 more
E/Zygote  ( 8430):      Suppressed: java.lang.ClassNotFoundException: Didn't find class &quot;com.android.server.SystemServer&quot; on path: DexPathList[[directory &quot;.&quot;],nativeLibraryDirectories=[/vendor/lib, /system/lib]]
E/Zygote  ( 8430):              at dalvik.system.BaseDexClassLoader.findClass(BaseDexClassLoader.java:56)
E/Zygote  ( 8430):              at java.lang.ClassLoader.loadClass(ClassLoader.java:511)
E/Zygote  ( 8430):              at java.lang.ClassLoader.loadClass(ClassLoader.java:504)
E/Zygote  ( 8430):              ... 9 more
E/Zygote  ( 8430):              Suppressed: java.lang.ClassNotFoundException: com.android.server.SystemServer
E/Zygote  ( 8430):                      at java.lang.Class.classForName(Native Method)
E/Zygote  ( 8430):                      at java.lang.BootClassLoader.findClass(ClassLoader.java:781)
E/Zygote  ( 8430):                      at java.lang.BootClassLoader.loadClass(ClassLoader.java:841)
E/Zygote  ( 8430):                      at java.lang.ClassLoader.loadClass(ClassLoader.java:504)
E/Zygote  ( 8430):                      ... 10 more
E/Zygote  ( 8430):              Caused by: java.lang.NoClassDefFoundError: Class not found using the boot class loader; no stack available
D/AndroidRuntime( 8430): Shutting down VM
E/AndroidRuntime( 8430): *** FATAL EXCEPTION IN SYSTEM PROCESS: main
E/AndroidRuntime( 8430): java.lang.RuntimeException: Missing class when invoking static main com.android.server.SystemServer
E/AndroidRuntime( 8430):        at com.android.internal.os.RuntimeInit.invokeStaticMain(RuntimeInit.java:204)
E/AndroidRuntime( 8430):        at com.android.internal.os.RuntimeInit.applicationInit(RuntimeInit.java:321)
E/AndroidRuntime( 8430):        at com.android.internal.os.RuntimeInit.zygoteInit(RuntimeInit.java:276)
E/AndroidRuntime( 8430):        at com.android.internal.os.ZygoteInit.handleSystemServerProcess(ZygoteInit.java:537)
E/AndroidRuntime( 8430):        at com.android.internal.os.ZygoteInit.startSystemServer(ZygoteInit.java:624)
E/AndroidRuntime( 8430):        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:686)
E/AndroidRuntime( 8430): Caused by: java.lang.ClassNotFoundException: com.android.server.SystemServer
E/AndroidRuntime( 8430):        at java.lang.Class.classForName(Native Method)
E/AndroidRuntime( 8430):        at java.lang.Class.forName(Class.java:308)
E/AndroidRuntime( 8430):        at com.android.internal.os.RuntimeInit.invokeStaticMain(RuntimeInit.java:202)
E/AndroidRuntime( 8430):        ... 5 more
E/AndroidRuntime( 8430): Caused by: java.lang.ClassNotFoundException: Didn't find class &quot;com.android.server.SystemServer&quot; on path: DexPathList[[zip file &quot;/system/framework/services.jar&quot;, zip file &quot;/system/framework/ethernet-service.jar&quot;, zip file &quot;/system/framework/wifi-service.jar&quot;],nativeLibraryDirectories=[/vendor/lib, /system/lib]]
E/AndroidRuntime( 8430):        at dalvik.system.BaseDexClassLoader.findClass(BaseDexClassLoader.java:56)
E/AndroidRuntime( 8430):        at java.lang.ClassLoader.loadClass(ClassLoader.java:511)
E/AndroidRuntime( 8430):        at java.lang.ClassLoader.loadClass(ClassLoader.java:469)
E/AndroidRuntime( 8430):        ... 8 more
E/AndroidRuntime( 8430):        Suppressed: java.io.IOException: Zip archive '/system/framework/services.jar' doesn't contain classes.dex (error msg: Entry not found)
E/AndroidRuntime( 8430):                at dalvik.system.DexFile.openDexFileNative(Native Method)
E/AndroidRuntime( 8430):                at dalvik.system.DexFile.openDexFile(DexFile.java:295)
E/AndroidRuntime( 8430):                at dalvik.system.DexFile.&lt;init&gt;(DexFile.java:80)
E/AndroidRuntime( 8430):                at dalvik.system.DexFile.&lt;init&gt;(DexFile.java:59)
E/AndroidRuntime( 8430):                at dalvik.system.DexPathList.loadDexFile(DexPathList.java:262)
E/AndroidRuntime( 8430):                at dalvik.system.DexPathList.makeDexElements(DexPathList.java:231)
E/AndroidRuntime( 8430):                at dalvik.system.DexPathList.&lt;init&gt;(DexPathList.java:109)
E/AndroidRuntime( 8430):                at dalvik.system.BaseDexClassLoader.&lt;init&gt;(BaseDexClassLoader.java:48)
E/AndroidRuntime( 8430):                at dalvik.system.PathClassLoader.&lt;init&gt;(PathClassLoader.java:38)
E/AndroidRuntime( 8430):                at com.android.internal.os.ZygoteInit.handleSystemServerProcess(ZygoteInit.java:530)
E/AndroidRuntime( 8430):                ... 2 more
E/AndroidRuntime( 8430):        Caused by: java.io.IOException: Failed to open oat file from dex location '/system/framework/services.jar'
E/AndroidRuntime( 8430):                ... 12 more
E/AndroidRuntime( 8430):        Caused by: java.io.IOException: Failed to open oat file from /system/framework/arm/services.odex (error Invalid oat magic for '/system/framework/arm/services.odex') (no dalvik_cache availible) and relocation failed.
E/AndroidRuntime( 8430):                ... 12 more
E/AndroidRuntime( 8430):        Caused by: java.io.IOException:
E/AndroidRuntime( 8430):                ... 12 more
E/AndroidRuntime( 8430):        Suppressed: java.lang.ClassNotFoundException: Didn't find class &quot;com.android.server.SystemServer&quot; on path: DexPathList[[directory &quot;.&quot;],nativeLibraryDirectories=[/vendor/lib, /system/lib]]
E/AndroidRuntime( 8430):                at dalvik.system.BaseDexClassLoader.findClass(BaseDexClassLoader.java:56)
E/AndroidRuntime( 8430):                at java.lang.ClassLoader.loadClass(ClassLoader.java:511)
E/AndroidRuntime( 8430):                at java.lang.ClassLoader.loadClass(ClassLoader.java:504)
E/AndroidRuntime( 8430):                ... 9 more
E/AndroidRuntime( 8430):                Suppressed: java.lang.ClassNotFoundException: com.android.server.SystemServer
E/AndroidRuntime( 8430):                        at java.lang.Class.classForName(Native Method)
E/AndroidRuntime( 8430):                        at java.lang.BootClassLoader.findClass(ClassLoader.java:781)
E/AndroidRuntime( 8430):                        at java.lang.BootClassLoader.loadClass(ClassLoader.java:841)
E/AndroidRuntime( 8430):                        at java.lang.ClassLoader.loadClass(ClassLoader.java:504)
E/AndroidRuntime( 8430):                        ... 10 more
E/AndroidRuntime( 8430):                Caused by: java.lang.NoClassDefFoundError: Class not found using the boot class loader; no stack available
E/AndroidRuntime( 8430): Error reporting crash
E/AndroidRuntime( 8430): java.lang.NullPointerException: Attempt to invoke interface method 'void android.app.IActivityManager.handleApplicationCrash(android.os.IBinder, android.app.ApplicationErrorReport$CrashInfo)' on a null object reference
E/AndroidRuntime( 8430):        at com.android.internal.os.RuntimeInit$UncaughtHandler.uncaughtException(RuntimeInit.java:89)
E/AndroidRuntime( 8430):        at java.lang.ThreadGroup.uncaughtException(ThreadGroup.java:693)
E/AndroidRuntime( 8430):        at java.lang.ThreadGroup.uncaughtException(ThreadGroup.java:690)
I/Process ( 8430): Sending signal. PID: 8430 SIG: 9
E/Zygote  ( 8205): Exit zygote because system server (8430) has terminated
[/shell]

彻底没辙了，看样子还有待深入研究，暂时只能先恢复吧，不然没手机用了，此时adb shell进去，发现su执行不生效，心中咯噔一下，不好，难道是要重刷系统，oh，不！最后还好连蒙带猜进到TWRP recovery(装[Multirom](http://forum.xda-developers.com/google-nexus-5/orig-development/mod-multirom-v24-t2571011)自带的)模式，挂载system分区，adb shell进去就已经是root权限，于是恢复之前备份的系统文件（这里可以看出备份重要文件的重要性了！！），还得注意一下权限是不是644，权限不对也进不了系统。最终终于进去系统了。

写这篇文章主要是记录这些错误信息，方便后面查阅，并且也为有类似想法的做次小白鼠，证明这条路暂时走不通！！！