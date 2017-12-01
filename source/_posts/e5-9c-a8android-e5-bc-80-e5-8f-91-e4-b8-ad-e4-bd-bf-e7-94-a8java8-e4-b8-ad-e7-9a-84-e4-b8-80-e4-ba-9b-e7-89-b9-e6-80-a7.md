---
title: 在Android开发中使用Java8中的一些特性——lambada expression
tags:
  - android
  - gradle-retrolambda
  - java8
  - lambada
  - retrolambda
id: 753
categories:
  - 学习
date: 2015-05-05 12:59:55
---

最近在学习Java8的一些新特性，感觉有很多非常实用的东西，如lambada expression，function interface、stream、property、binding等等，如果可以把这些特性（还有Java7的一些特性如try-resources-catch、Files/Path API）带入到Android开发中，那开发效率就会提高不少！<!--more-->

所以上网查了一下在Android开发中是否可以用Java8的一些特性，Android虽然主要使用Java语言开发，但其运行时的库和Java SE里的不一样，只是两者的API设计几乎一模一样，从Android 4.4开始Android才开始支持Java7，但Android自己的Java类库并没有完全实现Java SE 7。
所以如果想让使用了Java8的一些特性的代码在Java6、7下运行，一般是使用Java8编译，然后修改字节码转换成Java6、7的格式或使用低版本实现高版本的特性。

[retrolambda](https://github.com/orfjackal/retrolambda#getting-started)是一个运行Java 8带有lambada表达式在Java7或更低版本的类库
[gradle-retrolambda](https://github.com/evant/gradle-retrolambda)是一个封装了以上类库提供的一个gradle插件

我们今天的主题就是使用[gradle-retrolambda](https://github.com/evant/gradle-retrolambda)来实现在Android代码里实现lambada表达式
[这里](http://stackoverflow.com/questions/23318109/is-it-possible-to-use-java-8-for-android-development)参考了怎样在eclipse中使用，配置有些麻烦，但验证是可行的。
请仔细参考原文，这里只是提供几个优化过的脚本以及配置文件，可以在最新的ADT下运行。

gradle_build.cmd
```shell
@echo off
echo Starting the build %1 process...
gradle assembleDebug
pause
```

gradle_clean.cmd
```shell
@echo off
echo Cleaning the %1 project...
gradle clean
pause
```

gradle_post_build.cmd
```shell
@echo off

if exist build\outputs\apk\%1-debug-unaligned.apk (
	copy AndroidManifest.xml bin\AndroidManifest.xml
	copy build\outputs\apk\%1-debug-unaligned.apk bin\%1.apk
) else (
	echo Build project %1 error. Check the problems window for details...
)
```

builder配置如下图所示
[![eclipse_android_gradle](/resources/2015/05/eclipse_android_gradle.png)
](/resources/2015/05/eclipse_android_gradle.png)

原文中提供的build.gradle有些陈旧，这里提供一个较新的
build.gradle
```java
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.1.0'
     	classpath 'me.tatarka:gradle-retrolambda:3.1.0'
    }
}

// Required because retrolambda is on maven central
repositories {
  mavenCentral()
}

apply plugin: 'android'
apply plugin: 'me.tatarka.retrolambda'

dependencies {
    compile fileTree(dir: 'libs', include: '*.jar')
    compile 'com.jakewharton:butterknife:6.1.0'
}

task wrapper(type: Wrapper) {
    gradleVersion = '2.3'
}

android {
    compileSdkVersion 22
    buildToolsVersion '21.0.2'

    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
        }

        // Move the tests to tests/java, tests/res, etc...
        instrumentTest.setRoot('tests')

        // Move the build types to build-types/<type>
        // For instance, build-types/debug/java, build-types/debug/AndroidManifest.xml, ...
        // This moves them out of them default location under src/<type>/... which would
        // conflict with src/ being used by the main source set.
        // Adding new build types or product flavors should be accompanied
        // by a similar customization.
        debug.setRoot('build-types/debug')
        release.setRoot('build-types/release')
    }
}

```

如果使用的是Android Studio几乎就不需要什么多余的操作，只需要在build.gradle中添加classpath 'me.tatarka:gradle-retrolambda:3.1.0'的dependence以及apply plugin: 'me.tatarka.retrolambda'即可

这里之所以想在Android中使用Java8中lambada表达式是因为有时候在一些需要使用反射的情况下，可以使用lambada改善一下运行效率，即使在Java SE中也比MethodHandler的效率好，可参考[faster-alternatives-to-javas-reflection](http://stackoverflow.com/questions/19557829/faster-alternatives-to-javas-reflection)、[is-java-reflection-slow](https://vaskoz.wordpress.com/2013/07/15/is-java-reflection-slow/)