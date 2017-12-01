---
title: Android中support库的版本
tags:
  - android
  - android_support
id: 326
categories:
  - 学习
date: 2014-10-24 12:37:37
---

随着Android 5.0 SDK正式版的发布，Android里的support library也有了较大更新，在[官方文档](https://developer.android.com/tools/support-library/features.html)中v4、v7等库的依赖标识如下（其中版本号为21.0的为更新或新添加）：<!--more-->
[v4 Support Library](https://developer.android.com/tools/support-library/features.html#v4)
com.android.support:support-v4:**21.0.+
**[v7 Support Library](https://developer.android.com/tools/support-library/features.html#v7)**
**com.android.support:appcompat-v7:**21.0.+**
com.android.support:cardview-v7:**21.0.+**
com.android.support:gridlayout-v7:**21.0.+**
com.android.support:mediarouter-v7:**21.0.+**
com.android.support:palette-v7:**21.0.+**
com.android.support:recyclerview-v7:**21.0.+**
[v8 Support Library](https://developer.android.com/tools/support-library/features.html#v8)
[v13 Support Library](https://developer.android.com/tools/support-library/features.html#v13)
com.android.support:support-v13:18.0.+
[v17 Support Library](https://developer.android.com/tools/support-library/features.html#v17-leanback)
com.android.support:leanback-v17:**21.0.+**

这些库所支持的版本详见SDK一下目录内容%ANDROID_SDK_HOME%\extras\android\m2repository\com\android\support

[shell gutter="false" highlight="7,9,17,19,26,28,30,34,42,50"]
├───appcompat-v7
│ ├───18.0.0
│ ├───19.0.0
│ ├───19.0.1
│ ├───19.1.0
│ ├───20.0.0
│ └───21.0.0-rc1
├───cardview-v7
│ └───21.0.0-rc1
├───gridlayout-v7
│ ├───13.0.0
│ ├───18.0.0
│ ├───19.0.0
│ ├───19.0.1
│ ├───19.1.0
│ ├───20.0.0
│ └───21.0.0-rc1
├───leanback-v17
│ └───21.0.0-rc1
├───mediarouter-v7
│ ├───18.0.0
│ ├───19.0.0
│ ├───19.0.1
│ ├───19.1.0
│ ├───20.0.0
│ └───21.0.0-rc1
├───palette-v7
│ └───21.0.0-rc1
├───recyclerview-v7
│ └───21.0.0-rc1
├───support-annotations
│ ├───19.1.0
│ ├───20.0.0
│ └───21.0.0-rc1
├───support-v13
│ ├───13.0.0
│ ├───18.0.0
│ ├───19.0.0
│ ├───19.0.1
│ ├───19.1.0
│ ├───20.0.0
│ └───21.0.0-rc1
└───support-v4
 ├───13.0.0
 ├───18.0.0
 ├───19.0.0
 ├───19.0.1
 ├───19.1.0
 ├───20.0.0
 └───21.0.0-rc1
[/shell]

今天在使用com.android.support:support-v4:**21.0.+**时遇到一个问题

[shell gutter="false"]
Execution failed for task ':app:processDebugManifest'.
&gt; Manifest merger failed : uses-sdk:minSdkVersion 14 cannot be smaller than version L declared in library com.android.support:appcompat-v7:21.0.0-rc1
[/shell]

此错误的含义是指appcompat-v7:21.0.0-rc1这个库要求minSdkVersion为L，这似乎不合常理，因为support库本身就是为了在低版本下支持后来高版本才出现的一些特性，查看appcompat-v7:21.0.0-rc1这个库中定义的minSdkVersion（在%ANDROID_SDK_HOME%\extras\android\m2repository\com\android\support\appcompat-v7\21.0.0-rc1\appcompat-v7-21.0.0-rc1.aar 这个zip文件中AndroidManifest.xml有android:minSdkVersion="L"
），顺便看一下上一个版本appcompat-v7-20.0.0.aar的android:minSdkVersion="7"为预期正常的，再联想到这个版本号是rc版本，还不是正式版，所以可能这是一个小bug，应该在正式版中就会解决了。

<del>但目前所有使用更新过的21.0版本（实际是21.0.0-rc1，见以上高亮显示的版本号）的support库都会存在这个问题，即一旦引用了如com.android.support:support-v4:**21.0.+**都会要求应用的android:minSdkVersion至少为"L"，这显然是有问题的，目前临时的解决方法有以下几种</del>

<del>1\. 在AndroidManifest.xml中manifest标签中添加xmlns:tools="http://schemas.android.com/tools"，然后再添加或修改&lt;uses-sdk tools:node="replace" /&gt;</del>

<del>2\. 暂时不用最新的21.0版本，使用之前的一个版本，如com.android.support:appcompat-v7:**20.0.+**，这种方式会有一个警告如下图所示</del>
<del> [![support_appcompat_20_warning](http://202.203.209.55:8080/wp-content/uploads/2014/10/support_appcompat_20_warning.png)](http://202.203.209.55:8080/wp-content/uploads/2014/10/support_appcompat_20_warning.png)</del>

<del>3\. 删除如%ANDROID_SDK_HOME%\extras\android\m2repository\com\android\support\appcompat-v7\maven-metadata.xml中的&lt;version&gt;21.0.0-rc1&lt;/version&gt;</del>

<span style="color: #cc99ff;">_**目前Android Support Repository已更新至reversion 7解决此问题，21.0.0-rc1已更新为21.0.0**_</span>

这里备注一下版本号中"+"的含义
1\. com.android.support:support-v4:+ 不带版本号直接使用+表示最新版本，目前匹配21.0.0
2\. com.android.support:support-v4:19.+ 表示19.x.y...这类版本，目前匹配19.1.0
3\. **com.android.support:support-v4:19+ 与19.+含义一样，目前匹配19.1.0 (不常用，一般只用VersionNumber.+形式)**
4\. com.android.support:support-v4:19.0.+ 表示19.0.y...这类版本，目前匹配19.0.1
5\. com.android.support:support-v4:19.0 表示直接匹配19.0.1
不建议使用1方式，使用2、3方式的好处就是如果后面有bug修复版本可以自动升级使用最新的修复版本

还可以参考一下文档
[manifest-merger-failed-uses-sdkminsdkversion-14](http://stackoverflow.com/questions/24438170/manifest-merger-failed-uses-sdkminsdkversion-14#answer-24452251)
[add-cardview-to-android-os-versions-below-l](http://www.myandroidsolutions.com/2014/08/11/add-cardview-to-android-os-versions-below-l/)