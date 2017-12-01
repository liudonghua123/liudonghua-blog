---
title: Android 5.1的升级过程
id: 690
categories:
  - 学习
date: 2015-03-12 15:23:10
tags:
---

最近Android系统有5.1更新了，所以就想很快的用上，factory images刷机倒是很简单，但手机上有很多数据，不想重装那么麻烦，所以就只能手动安装OTA升级包了。<!--more-->

这其中遇到很多问题，都是由于我之前系统被我用CF-Auto-Root修改过，也就是root过，并且这个root工具没有恢复的方法，在网上搜了很久一般给出的解决方法都是wipe cache + (factory reset)或者直接重刷，这样都会丢失已有数据。

所以我就尝试手动恢复被CF-Auto-Root修改过文件，但CF-Auto-Root-hammerhead-hammerhead-nexus5.img文件有不同Android中的system.img、radio.img格式，无法用ext4_unpacker+ext2explore打开查看，所以只能另辟蹊径，通过手动升级OTA包时报的错来查看那些文件被修改了，然后提取image-hammerhead-lrx22c.zip中的system.img中的文件进行恢复。

第一次OTA升级时主要提示如下错误

[shell highlight="18"]
Now send the package you want to apply
to the device with &quot;adb sideload &lt;filename&gt;&quot;...
Starting to open usb_init()
unix_open to open usb_init(): -1
sideload-host file size 231406578 block size 65536
Finding update package...
I:Update location: /sideload/package.zip
Opening update package...
I:read key e=65537 hash=20
I:read key e=65537 hash=20
I:2 key(s) loaded from /res/keys
Verifying update package...
I:comment is 1370 bytes; signature 1352 bytes from end
I:whole-file signature verified against RSA key 0
I:verify_file returned 0
Installing update...
Verifying current system...
file &quot;/system/bin/install-recovery.sh&quot; doesn't have any of expected sha1 sums; checking cache
failed to stat &quot;/cache/saved.file&quot;: No such file or directory
failed to load cache file
script aborted: &quot;/system/bin/install-recovery.sh&quot; has unexpected contents.
&quot;/system/bin/install-recovery.sh&quot; has unexpected contents.
E:Error in /sideload/package.zip
(Status 7)
[/shell]

查看升级包中updater-script的有如下片段

[shell]
apply_patch_check(&quot;/system/bin/install-recovery.sh&quot;, &quot;fe314eac1400ea76e9e7cfef55ed1779663e848a&quot;, &quot;53b2a90b71a9b73f3857bb3ea0d3dc0cbac481b5&quot;) || abort(&quot;\&quot;/system/bin/install-recovery.sh\&quot; has unexpected contents.&quot;);
[/shell]

并且提取lrx22c的system.img中的install-recovery.sh检查sha1符合53B2A90B71A9B73F3857BB3EA0D3DC0CBAC481B5，所以用之替换掉现有的。

再次OTA升级又有如下错误

[shell highlight="20,21"]
Now send the package you want to apply
to the device with &quot;adb sideload &lt;filename&gt;&quot;...
Starting to open usb_init()
unix_open to open usb_init(): -1
sideload-host file size 231406578 block size 65536
Finding update package...
I:Update location: /sideload/package.zip
Opening update package...
I:read key e=65537 hash=20
I:read key e=65537 hash=20
I:2 key(s) loaded from /res/keys
Verifying update package...
I:comment is 1370 bytes; signature 1352 bytes from end
I:whole-file signature verified against RSA key 0
I:verify_file returned 0
Installing update...
Verifying current system...
failed to stat &quot;system/priv-app/Hangouts/Hangouts.apk&quot;: No such file or directory
failed to stat &quot;system/priv-app/Hangouts/lib/arm/libvideochat_jni.so&quot;: No such file or directory
contents of partition &quot;/dev/block/platform/msm_sdcc.1/by-name/boot&quot; didn't match EMMC:/dev/block/platform/msm_sdcc.1/by-name/boot:9064448:98295c341828085ec8722608a27a112ea9057ece:9144320:b0c48c1d1c9c0c1434804e127d2b9dfc94cc93c0
file &quot;EMMC:/dev/block/platform/msm_sdcc.1/by-name/boot:9064448:98295c341828085ec8722608a27a112ea9057ece:9144320:b0c48c1d1c9c0c1434804e127d2b9dfc94cc93c0&quot; doesn't have any of expected sha1 sums; checking cache
failed to stat &quot;/cache/saved.file&quot;: No such file or directory
failed to load cache file
script aborted: &quot;EMMC:/dev/block/platform/msm_sdcc.1/by-name/boot:9064448:98295c341828085ec8722608a27a112ea9057ece:9144320:b0c48c1d1c9c0c1434804e127d2b9dfc94cc93c0&quot; has unexpected contents.
&quot;EMMC:/dev/block/platform/msm_sdcc.1/by-name/boot:9064448:98295c341828085ec8722608a27a112ea9057ece:9144320:b0c48c1d1c9c0c1434804e127d2b9dfc94cc93c0&quot; has unexpected contents.
E:Error in /sideload/package.zip
(Status 7)
[/shell]

查看升级包中updater-script的有如下片段

[shell]
apply_patch_check(&quot;EMMC:/dev/block/platform/msm_sdcc.1/by-name/boot:9064448:98295c341828085ec8722608a27a112ea9057ece:9144320:b0c48c1d1c9c0c1434804e127d2b9dfc94cc93c0&quot;) || abort(&quot;\&quot;EMMC:/dev/block/platform/msm_sdcc.1/by-name/boot:9064448:98295c341828085ec8722608a27a112ea9057ece:9144320:b0c48c1d1c9c0c1434804e127d2b9dfc94cc93c0\&quot; has unexpected contents.&quot;);
[/shell]

虽然不清楚EMMC:/dev/block/platform/msm_sdcc.1/by-name/boot:9064448:98295c341828085ec8722608a27a112ea9057ece:9144320:b0c48c1d1c9c0c1434804e127d2b9dfc94cc93c0"是什么，但由于CF-Auto-Root会修改boot分区，所以猜测这应该是boot分区对应的文件或挂载点，所以重刷lrx22c的boot.img。

最后再使用OTA升级，终于成功了~~~

附 手动安装OTA的方法
1\. 下载对应自己系统的升级包
2\. 重启手机到recovery模式
3\. 选择"_apply update from adb_ "
4\. PC端使用"adb sideload **_your_ota_package_**.zip"
5\. 升级错误日志见/cache/recovery/last_log(.&lt;num&gt;)，其中时间顺序最近依次是last_log、last_log.1、last_log.2、......

这次root时就不用CF-Auto-Root了，还是规规矩矩在CMW Recovery下刷[UPDATE-SuperSU](http://download.chainfire.eu/696/SuperSU/UPDATE-SuperSU-v2.46.zip?retrieve_file=1)吧！这样至少install-recovery.sh会备份并且知道它安装的过程，修改了哪些文件！

<!--more-->参考资料
1\. [How to Extract system.img File or Get System Dump of Android Devices on Windows](http://www.droidviews.com/extract-system-img-files-or-system-dump-of-android-devices-on-windows/)
2\. [CF-Auto-Root](http://autoroot.chainfire.eu/)
3. [How to manually install the Lollipop OTA on your Nexus device, download links included [updated with 5.1 links]](http://www.talkandroid.com/guides/beginner/how-to-install-the-lollipop-ota-on-your-nexus-device-download-links-included/)
4. [ext4_unpacker+ext2explore](http://202.203.209.55:8080/wp-content/uploads/2015/03/ext4_unpacker-ext2explore.zip)