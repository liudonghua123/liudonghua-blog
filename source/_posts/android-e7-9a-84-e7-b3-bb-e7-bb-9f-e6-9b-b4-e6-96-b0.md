---
title: Android的系统更新
tags:
  - android
id: 679
categories:
  - 学习
date: 2015-03-11 10:05:33
---

昨天看到消息Google发布了Android 5.1(Codename还是Lollipop，说明变化不大)，但是上网搜了一下只有factory image，这样刷的话会清除手机上原来的所有数据，而且我原来装的系统也没有被我修改过，所以最好还是通过OTA升级，但OTA升级在国内一般都比较慢，而且由于Google在国内不稳定或根本就无法访问，所以最好还是手动下载OTA升级。<!--more-->

但找了很久都没找到N5的OTA下载链接，可能过几天就会有了。

所以就想看一下Android“系统更新”模块是怎样实现的，这样或许就可以直接从代码里找出下载链接

在Settings的源码下一步一步搜索，最终找到这里
res\xml\device_info_settings.xml

[xml]
        &lt;!-- System update settings - launches activity --&gt;
        &lt;PreferenceScreen android:key=&quot;system_update_settings&quot;
                android:title=&quot;@string/system_update_settings_list_item_title&quot;
                android:summary=&quot;@string/system_update_settings_list_item_summary&quot;&gt;
            &lt;intent android:action=&quot;android.settings.SYSTEM_UPDATE_SETTINGS&quot; /&gt;
        &lt;/PreferenceScreen&gt;
[/xml]

也就是说在设置里点了系统更新之后，只会发一个"android.settings.SYSTEM_UPDATE_SETTINGS"的广播，然后由注册了这个广播的组件来处理。
GoogleServicesFramework的AndroidManifest.xml有

[xml]
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot; standalone=&quot;no&quot;?&gt;
&lt;manifest xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot; android:sharedUserId=&quot;com.google.uid.shared&quot; package=&quot;com.google.android.gsf&quot; platformBuildVersionCode=&quot;21&quot; platformBuildVersionName=&quot;5.0.1-1602158&quot;&gt;
    &lt;application&gt;
        ......
        &lt;activity android:enabled=&quot;false&quot; android:hardwareAccelerated=&quot;true&quot; android:label=&quot;@string/system_update_activity_name&quot; android:launchMode=&quot;singleTop&quot; android:name=&quot;.update.SystemUpdateActivity&quot; android:theme=&quot;@android:style/Theme.Holo.NoActionBar&quot;&gt;
            &lt;intent-filter&gt;
                &lt;action android:name=&quot;android.settings.SYSTEM_UPDATE_SETTINGS&quot;/&gt;
                &lt;category android:name=&quot;android.intent.category.DEFAULT&quot;/&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;
        ......
    &lt;/application&gt;
&lt;/manifest&gt;
[/xml]

对于N5来说，就是由GoogleServicesFramework中的com.google.android.gsf.update.SystemUpdateActivity来处理，由于这部分源码不是开源的，反编译代码错综复杂，现在暂时还没找到。

不过后来搜索以前N5的OTA下载地址，网址一般都是http://android.clients.google.com/packages/ota/**google_hammerhead**/**some_hash**.signed-hammerhead-**new_build_name**-from-**old_build_name**.**some_other_hash**.zip
如Nexus 5 (hammerhead) LRX21O to LRX22C
http://android.clients.google.com/packages/ota/**google_hammerhead**/**785a2f7af3718dba7e569decde8b6c4dc476a309**.signed-hammerhead-**LRX22C**-from-**LRX21O**.**785a2f7a**.zip

<del>这样在Google中搜索"google_hammerhead LMY47D LRX22C"关键字，找到如下的下载链接</del>
<del> http://android.clients.google.com/packages/ota/google_hammerhead/683daf65f4daf477b5f42f565f42f42f5681b03f41.signed-hammerhead-1234567890123LMY47D-from-LRX22C.683daf65.zip</del>
<del> 现在正在验证一下是不是可以下载。
</del>已证实上面地址是假的，在EC2服务器上下载时404

2015/3/21 更新：正确的下载地址  http://android.clients.google.com/packages/ota/google_hammerhead/e16268c5a3df75a65054ab258d7d615288b7e3b4.signed-hammerhead-ota-LMY47D-from-LRX22C-radio-restricted.zip ， 详见[Download: Android 5.1 OTA Updates for Nexus Devices (Updated)](http://www.droid-life.com/2015/03/11/download-android-5-1-ota-updates-for-nexus-devices-4-5-6-7-9-10/)