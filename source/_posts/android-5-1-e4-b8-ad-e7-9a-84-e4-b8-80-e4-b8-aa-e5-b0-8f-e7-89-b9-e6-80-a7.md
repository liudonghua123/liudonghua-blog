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

这里给大家介绍一个很小很小（小到很多人可能都不会注意到，哈哈~~~）的改进。
如下图所示
[![android-5.1-deskclock-anims](http://202.203.209.55:8080/wp-content/uploads/2015/03/android-5.1-deskclock-anims.gif)](http://202.203.209.55:8080/wp-content/uploads/2015/03/android-5.1-deskclock-anims.gif)
不细看代码可能认为不就是把原来的png改成了gif吧，其实不是这样的。
这个小的修改是使用了objectAnimator动画以及vector位图，就以第一个闹钟图标来说。
其动画是由下面几个部分组成！（详细这个小特性的修改见[Animating tab icons](https://android.googlesource.com/platform/packages/apps/DeskClock/+/3f49d162e72b227fb6fe142a06ba0deacca79200%5E1..3f49d162e72b227fb6fe142a06ba0deacca79200/)）
[su_accordion]
[su_spoiler title="ic_alarm_animation_button.xml" icon="arrow-circle-1"]

[xml]
&lt;pre&gt;&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;set xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot; &gt;
    &lt;set
        android:ordering=&quot;sequentially&quot; &gt;
        &lt;objectAnimator
            android:duration=&quot;33&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;0&quot;
            android:valueTo=&quot;8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;8&quot;
            android:valueTo=&quot;-8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;-8&quot;
            android:valueTo=&quot;8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;8&quot;
            android:valueTo=&quot;-8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;-8&quot;
            android:valueTo=&quot;8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;8&quot;
            android:valueTo=&quot;-8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;-8&quot;
            android:valueTo=&quot;8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;8&quot;
            android:valueTo=&quot;-8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;-8&quot;
            android:valueTo=&quot;8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;8&quot;
            android:valueTo=&quot;-8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;-8&quot;
            android:valueTo=&quot;8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;8&quot;
            android:valueTo=&quot;-8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;-8&quot;
            android:valueTo=&quot;8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;8&quot;
            android:valueTo=&quot;-8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;-8&quot;
            android:valueTo=&quot;8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;8&quot;
            android:valueTo=&quot;-8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;-8&quot;
            android:valueTo=&quot;8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;8&quot;
            android:valueTo=&quot;-8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;-8&quot;
            android:valueTo=&quot;8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;67&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;8&quot;
            android:valueTo=&quot;-8&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
        &lt;objectAnimator
            android:duration=&quot;33&quot;
            android:propertyName=&quot;rotation&quot;
            android:valueFrom=&quot;-8&quot;
            android:valueTo=&quot;0&quot;
            android:interpolator=&quot;@android:interpolator/fast_out_slow_in&quot; /&gt;
    &lt;/set&gt;
&lt;/set&gt;
[/xml]

[/su_spoiler]

[su_spoiler title="ic_alarm_animation.xml" icon="arrow-circle-1"]

[xml]
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;animated-selector xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;&gt;

    &lt;item android:state_selected=&quot;false&quot; android:state_focused=&quot;true&quot;
        android:drawable=&quot;@drawable/ic_tab_alarm_activated&quot; /&gt;

    &lt;item android:id=&quot;@+id/on&quot; android:state_selected=&quot;true&quot;
        android:drawable=&quot;@drawable/ic_tab_alarm_activated&quot; /&gt;

    &lt;item android:id=&quot;@+id/off&quot;
        android:drawable=&quot;@drawable/ic_tab_alarm_normal&quot; /&gt;

    &lt;transition android:fromId=&quot;@id/off&quot; android:toId=&quot;@id/on&quot;&gt;
        &lt;animated-vector android:drawable=&quot;@drawable/ic_alarm&quot;&gt;
            &lt;target
                android:name=&quot;button&quot;
                android:animation=&quot;@animator/ic_alarm_animation_button&quot; /&gt;
        &lt;/animated-vector&gt;
    &lt;/transition&gt;

&lt;/animated-selector&gt;
[/xml]

[/su_spoiler]

[su_spoiler title="ic_alarm.xml" icon="arrow-circle-1"]

[xml]
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;vector xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;
    android:height=&quot;24dp&quot;
    android:width=&quot;24dp&quot;
    android:viewportHeight=&quot;24&quot;
    android:viewportWidth=&quot;24&quot; &gt;
    &lt;group
        android:name=&quot;button&quot;
        android:translateX=&quot;12&quot;
        android:translateY=&quot;12&quot; &gt;
        &lt;group
            android:name=&quot;button_pivot&quot;
            android:translateX=&quot;-12&quot;
            android:translateY=&quot;-12&quot; &gt;
            &lt;group
                android:name=&quot;right_button&quot;
                android:translateX=&quot;19.0722&quot;
                android:translateY=&quot;4.5758&quot; &gt;
                &lt;path
                    android:name=&quot;path_1&quot;
                    android:pathData=&quot;M 2.9409942627,1.16249084473 c 0.0,0.0 -4.59599304199,-3.85699462891 -4.59599304199,-3.85699462891 c 0.0,0.0 -1.2859954834,1.53300476074 -1.2859954834,1.53300476074 c 0.0,0.0 4.59599304199,3.85600280762 4.59599304199,3.85600280762 c 0.0,0.0 1.2859954834,-1.53201293945 1.2859954834,-1.53201293945 Z&quot;
                    android:fillColor=&quot;#FFFFFFFF&quot;
                    android:fillAlpha=&quot;1&quot; /&gt;
            &lt;/group&gt;
            &lt;group
                android:name=&quot;left_button&quot;
                android:translateX=&quot;4.9262&quot;
                android:translateY=&quot;4.5729&quot; &gt;
                &lt;path
                    android:name=&quot;left_button_path&quot;
                    android:pathData=&quot;M 2.9409942627,-1.16250610352 c 0.0,0.0 -1.2859954834,-1.53199768066 -1.2859954834,-1.53199768066 c 0.0,0.0 -4.59599304199,3.85600280762 -4.59599304199,3.85600280762 c 0.0,0.0 1.2859954834,1.53300476074 1.2859954834,1.53300476074 c 0.0,0.0 4.59599304199,-3.8570098877 4.59599304199,-3.8570098877 Z&quot;
                    android:fillColor=&quot;#FFFFFFFF&quot;
                    android:fillAlpha=&quot;1&quot; /&gt;
            &lt;/group&gt;
        &lt;/group&gt;
    &lt;/group&gt;
    &lt;group
        android:name=&quot;alarm&quot;
        android:translateX=&quot;12&quot;
        android:translateY=&quot;12&quot; &gt;
        &lt;group
            android:name=&quot;alarm_pivot&quot;
            android:translateX=&quot;-12&quot;
            android:translateY=&quot;-12&quot; &gt;
            &lt;group
                android:name=&quot;alarm_hands&quot;
                android:translateX=&quot;13.75&quot;
                android:translateY=&quot;12.4473&quot; &gt;
                &lt;path
                    android:name=&quot;alarm_hands_path&quot;
                    android:pathData=&quot;M -1.25,-4.42700195312 c 0.0,0.0 -1.5,0.0 -1.5,0.0 c 0.0,0.0 0.0,6.0 0.0,6.0 c 0.0,0.0 4.74699401855,2.85400390625 4.74699401855,2.85400390625 c 0.0,0.0 0.753005981445,-1.23199462891 0.753005981445,-1.23199462891 c 0.0,0.0 -4.0,-2.37200927734 -4.0,-2.37200927734 c 0.0,0.0 0.0,-5.25 0.0,-5.25 Z&quot;
                    android:fillColor=&quot;#FFFFFFFF&quot;
                    android:fillAlpha=&quot;1&quot; /&gt;
            &lt;/group&gt;
            &lt;group
                android:name=&quot;alarm_body&quot;
                android:translateX=&quot;12&quot;
                android:translateY=&quot;13.0203&quot; &gt;
                &lt;path
                    android:name=&quot;alarm_outline_path&quot;
                    android:pathData=&quot;M -0.0050048828125,-9 c -4.97399902344,0 -8.99499511719,4.0299987793 -8.99499511719,9 c 0,4.9700012207 4.02099609375,9 8.99499511719,9 c 4.97399902344,0 9.00500488281,-4.0299987793 9.00500488281,-9 c 0,-4.9700012207 -4.03100585938,-9 -9.00500488281,-9 Z M 0,7 c -3.86700439453,0 -7,-3.13400268555 -7,-7 c 0,-3.86599731445 3.13299560547,-7 7,-7 c 3.86700439453,0 7,3.13400268555 7,7 c 0,3.86599731445 -3.13299560547,7 -7,7 Z&quot;
                    android:fillColor=&quot;#FFFFFFFF&quot;
                    android:fillAlpha=&quot;1&quot; /&gt;
            &lt;/group&gt;
        &lt;/group&gt;
    &lt;/group&gt;
&lt;/vector&gt;
[/xml]

[/su_spoiler]
[/su_accordion]