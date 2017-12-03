---
title: Android 5.1中的一个小特性
tags:
  - android
  - animation
  - deskclock
id: 698
categories:
  - 学习
date: 2015-03-13 11:23:14
---

在新的Android 5.1中虽然没有什么特别大的特性，但还是有一些小特性。可详见[What's new in Android 5.1? Some big and small changes in the updated Lollipop release](http://www.androidcentral.com/whats-new-android-51-some-big-and-small-changes-updated-lollipop-release)。

<!--more-->

这里给大家介绍一个很小很小（小到很多人可能都不会注意到，哈哈~~~）的改进。
如下图所示
[![android-5.1-deskclock-anims](/resources/2015/03/android-5.1-deskclock-anims.gif)](/resources/2015/03/android-5.1-deskclock-anims.gif)
不细看代码可能认为不就是把原来的png改成了gif吧，其实不是这样的。
这个小的修改是使用了objectAnimator动画以及vector位图，就以第一个闹钟图标来说。
其动画是由下面几个部分组成！（详细这个小特性的修改见[Animating tab icons](https://android.googlesource.com/platform/packages/apps/DeskClock/+/3f49d162e72b227fb6fe142a06ba0deacca79200%5E1..3f49d162e72b227fb6fe142a06ba0deacca79200/)）
[su_accordion]
[su_spoiler title="ic_alarm_animation_button.xml" icon="arrow-circle-1"]

```xml
<pre><?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android" >
    <set
        android:ordering="sequentially" >
        <objectAnimator
            android:duration="33"
            android:propertyName="rotation"
            android:valueFrom="0"
            android:valueTo="8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="8"
            android:valueTo="-8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="-8"
            android:valueTo="8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="8"
            android:valueTo="-8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="-8"
            android:valueTo="8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="8"
            android:valueTo="-8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="-8"
            android:valueTo="8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="8"
            android:valueTo="-8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="-8"
            android:valueTo="8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="8"
            android:valueTo="-8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="-8"
            android:valueTo="8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="8"
            android:valueTo="-8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="-8"
            android:valueTo="8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="8"
            android:valueTo="-8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="-8"
            android:valueTo="8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="8"
            android:valueTo="-8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="-8"
            android:valueTo="8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="8"
            android:valueTo="-8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="-8"
            android:valueTo="8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="67"
            android:propertyName="rotation"
            android:valueFrom="8"
            android:valueTo="-8"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
        <objectAnimator
            android:duration="33"
            android:propertyName="rotation"
            android:valueFrom="-8"
            android:valueTo="0"
            android:interpolator="@android:interpolator/fast_out_slow_in" />
    </set>
</set>
```

[/su_spoiler]

[su_spoiler title="ic_alarm_animation.xml" icon="arrow-circle-1"]

```xml
<?xml version="1.0" encoding="utf-8"?>
<animated-selector xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:state_selected="false" android:state_focused="true"
        android:drawable="@drawable/ic_tab_alarm_activated" />

    <item android:id="@+id/on" android:state_selected="true"
        android:drawable="@drawable/ic_tab_alarm_activated" />

    <item android:id="@+id/off"
        android:drawable="@drawable/ic_tab_alarm_normal" />

    <transition android:fromId="@id/off" android:toId="@id/on">
        <animated-vector android:drawable="@drawable/ic_alarm">
            <target
                android:name="button"
                android:animation="@animator/ic_alarm_animation_button" />
        </animated-vector>
    </transition>

</animated-selector>
```

[/su_spoiler]

[su_spoiler title="ic_alarm.xml" icon="arrow-circle-1"]

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:height="24dp"
    android:width="24dp"
    android:viewportHeight="24"
    android:viewportWidth="24" >
    <group
        android:name="button"
        android:translateX="12"
        android:translateY="12" >
        <group
            android:name="button_pivot"
            android:translateX="-12"
            android:translateY="-12" >
            <group
                android:name="right_button"
                android:translateX="19.0722"
                android:translateY="4.5758" >
                <path
                    android:name="path_1"
                    android:pathData="M 2.9409942627,1.16249084473 c 0.0,0.0 -4.59599304199,-3.85699462891 -4.59599304199,-3.85699462891 c 0.0,0.0 -1.2859954834,1.53300476074 -1.2859954834,1.53300476074 c 0.0,0.0 4.59599304199,3.85600280762 4.59599304199,3.85600280762 c 0.0,0.0 1.2859954834,-1.53201293945 1.2859954834,-1.53201293945 Z"
                    android:fillColor="#FFFFFFFF"
                    android:fillAlpha="1" />
            </group>
            <group
                android:name="left_button"
                android:translateX="4.9262"
                android:translateY="4.5729" >
                <path
                    android:name="left_button_path"
                    android:pathData="M 2.9409942627,-1.16250610352 c 0.0,0.0 -1.2859954834,-1.53199768066 -1.2859954834,-1.53199768066 c 0.0,0.0 -4.59599304199,3.85600280762 -4.59599304199,3.85600280762 c 0.0,0.0 1.2859954834,1.53300476074 1.2859954834,1.53300476074 c 0.0,0.0 4.59599304199,-3.8570098877 4.59599304199,-3.8570098877 Z"
                    android:fillColor="#FFFFFFFF"
                    android:fillAlpha="1" />
            </group>
        </group>
    </group>
    <group
        android:name="alarm"
        android:translateX="12"
        android:translateY="12" >
        <group
            android:name="alarm_pivot"
            android:translateX="-12"
            android:translateY="-12" >
            <group
                android:name="alarm_hands"
                android:translateX="13.75"
                android:translateY="12.4473" >
                <path
                    android:name="alarm_hands_path"
                    android:pathData="M -1.25,-4.42700195312 c 0.0,0.0 -1.5,0.0 -1.5,0.0 c 0.0,0.0 0.0,6.0 0.0,6.0 c 0.0,0.0 4.74699401855,2.85400390625 4.74699401855,2.85400390625 c 0.0,0.0 0.753005981445,-1.23199462891 0.753005981445,-1.23199462891 c 0.0,0.0 -4.0,-2.37200927734 -4.0,-2.37200927734 c 0.0,0.0 0.0,-5.25 0.0,-5.25 Z"
                    android:fillColor="#FFFFFFFF"
                    android:fillAlpha="1" />
            </group>
            <group
                android:name="alarm_body"
                android:translateX="12"
                android:translateY="13.0203" >
                <path
                    android:name="alarm_outline_path"
                    android:pathData="M -0.0050048828125,-9 c -4.97399902344,0 -8.99499511719,4.0299987793 -8.99499511719,9 c 0,4.9700012207 4.02099609375,9 8.99499511719,9 c 4.97399902344,0 9.00500488281,-4.0299987793 9.00500488281,-9 c 0,-4.9700012207 -4.03100585938,-9 -9.00500488281,-9 Z M 0,7 c -3.86700439453,0 -7,-3.13400268555 -7,-7 c 0,-3.86599731445 3.13299560547,-7 7,-7 c 3.86700439453,0 7,3.13400268555 7,7 c 0,3.86599731445 -3.13299560547,7 -7,7 Z"
                    android:fillColor="#FFFFFFFF"
                    android:fillAlpha="1" />
            </group>
        </group>
    </group>
</vector>
```

[/su_spoiler]
[/su_accordion]