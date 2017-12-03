---
title: Android Eclipse环境搭建
tags:
  - android
  - eclipse
id: 168
categories:
  - 学习
date: 2014-10-16 14:32:37
---

这方面资料其实已经很多了，主要是安装maven支持的插件有时候会遇到一些问题，这里记录一些环境搭建中用到的文档资料以及曾经遇到过的问题以及解决方法

<!--more-->

1\. 简单的Android Eclipse环境搭建
可以下载Android官方提供的[Eclipse ADT](http://developer.android.com/sdk/index.html)，这个就是几乎帮你配置好的，包括Eclipse+ADT plugin+SDK
只是这里面自带的Eclipse版本不是最新的，SDK里面只包括最新版的Android支持，如我下载的支持4.4，如果想使用最新版的Eclipse搭建，则需要自己再下载[SDK](http://developer.android.com/sdk/installing/index.html?pkg=tools)、[ADT plugin](http://developer.android.com/sdk/installing/installing-adt.html)安装配置

2\. 支持Maven的环境搭建
<del>在上面的基础上还需要安装[m2e-android](http://rgladwell.github.io/m2e-android/)插件，这个插件我不管在Market place还是通过[update site](http://rgladwell.github.com/m2e-android/updates/)安装始终有问题，不能安装上去，从网上又找不到离线安装包，由于这个开源的，所以就自己编译安装吧！</del>
<del> 1\. 下载[m2e-android](https://github.com/rgladwell/m2e-android)代码</del>
<del> git clone https://github.com/rgladwell/m2e-android.git</del>
<del> 2\. 下载、安装[maven-android-sdk-deployer](https://github.com/mosabua/maven-android-sdk-deployer)，这个对于Android maven开发是非常重要的</del>
<del> git clone https://github.com/mosabua/maven-android-sdk-deployer.git</del>
<del> cd maven-android-sdk-deployer</del>
<del> mvn install -P 4.4     #注，这里需要配合你已安装的SDK Android支持版本</del>
<del> 3\. 编译安装[m2e-android](https://github.com/rgladwell/m2e-android)</del>
<del> mvn --file org.sonatype.aether/pom.xml install</del>
<del> mvn install</del>
<del>之后就编译出本地安装包</del>
<del>%USERPROFILE%\.m2\repository\me\gladwell\eclipse\m2e\android\me.gladwell.eclipse.m2e.android.update</del>
<del>~/.m2/repository/me/gladwell/eclipse/m2e/android/me.gladwell.eclipse.m2e.android.update</del>
<del>然后就可以本地安装</del>

mvn install -P 4.4 过程中可能会遇到的错误

[text]
 [ERROR] Failed to execute goal org.codehaus.mojo:properties-maven-plugin:1.0-alpha-2:read-project-properties (default) on project google-apis-19: Properties file not found: D:\android\android-studio\sdk\add-ons\addon-google_apis-google-19\source.properties -> [Help 1]
[/text]

是因为不存在add-ons\addon-google_apis-google-19，即没有安装Google APIs 19

安装[m2e-android](http://rgladwell.github.io/m2e-android/)插件时可能遇到的问题是提示如下错误，其实可以在安装时[Uncheck "Contact all update sites during install to find required software"](http://stackoverflow.com/questions/6470802/what-to-do-about-eclipses-no-repository-found-containing-error-messages) 就可以最终安装上去了

[text collapse="true" toolbar="true" 1="install" 2="error" 3="3="message"</code>"" 4="language="<code>title="m2e-android""]
An error occurred while collecting items to be installed
session context was:(profile=epp.package.jee, phase=org.eclipse.equinox.internal.p2.engine.phases.Collect, operand=, action=).
No repository found containing: osgi.bundle,org.eclipse.ecf,3.4.0.v20140827-1444
No repository found containing: osgi.bundle,org.eclipse.ecf.filetransfer,5.0.0.v20140827-1444
No repository found containing: osgi.bundle,org.eclipse.ecf.identity,3.4.0.v20140827-1444
No repository found containing: osgi.bundle,org.eclipse.ecf.provider.filetransfer,3.2.200.v20140827-1444
No repository found containing: osgi.bundle,org.eclipse.ecf.provider.filetransfer.httpclient4,1.0.800.v20140827-1444
No repository found containing: osgi.bundle,org.eclipse.ecf.provider.filetransfer.httpclient4.ssl,1.0.0.v20140827-1444
No repository found containing: osgi.bundle,org.eclipse.ecf.provider.filetransfer.ssl,1.0.0.v20140827-1444
No repository found containing: osgi.bundle,org.eclipse.ecf.ssl,1.1.0.v20140827-1444
No repository found containing: osgi.bundle,org.eclipse.emf,2.6.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.ant,2.8.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.codegen,2.10.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.codegen.ecore,2.10.1.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.codegen.ecore.ui,2.10.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.codegen.ui,2.6.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.common,2.10.1.v20140901-1043
No repository found containing: osgi.bundle,org.eclipse.emf.common.ui,2.9.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.converter,2.7.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.databinding,1.3.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.databinding.edit,1.3.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.ecore,2.10.1.v20140901-1043
No repository found containing: osgi.bundle,org.eclipse.emf.ecore.change,2.10.0.v20140901-1043
No repository found containing: osgi.bundle,org.eclipse.emf.ecore.change.edit,2.6.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.ecore.edit,2.9.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.ecore.editor,2.10.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.ecore.xmi,2.10.1.v20140901-1043
No repository found containing: osgi.bundle,org.eclipse.emf.edit,2.10.1.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.edit.ui,2.10.1.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.exporter,2.7.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.importer,2.9.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.importer.ecore,2.8.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.importer.java,2.7.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.importer.rose,2.7.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.mapping,2.8.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.mapping.ecore,2.6.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.mapping.ecore.editor,2.6.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.mapping.ecore2ecore,2.8.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.mapping.ecore2ecore.editor,2.7.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.mapping.ecore2xml,2.8.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.mapping.ecore2xml.ui,2.7.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.emf.mapping.ui,2.6.0.v20140901-1055
No repository found containing: osgi.bundle,org.eclipse.xsd,2.10.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.ecf.core.feature,1.1.0.v20140827-1444
No repository found containing: org.eclipse.update.feature,org.eclipse.ecf.core.ssl.feature,1.0.0.v20140827-1444
No repository found containing: org.eclipse.update.feature,org.eclipse.ecf.filetransfer.feature,3.9.0.v20140827-1444
No repository found containing: org.eclipse.update.feature,org.eclipse.ecf.filetransfer.httpclient4.feature,3.9.1.v20140827-1444
No repository found containing: org.eclipse.update.feature,org.eclipse.ecf.filetransfer.httpclient4.ssl.feature,1.0.0.v20140827-1444
No repository found containing: org.eclipse.update.feature,org.eclipse.ecf.filetransfer.ssl.feature,1.0.0.v20140827-1444
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.codegen.ecore,2.10.1.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.codegen.ecore.ui,2.10.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.codegen,2.10.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.codegen.ui,2.8.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.common,2.10.1.v20140901-1043
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.common.ui,2.9.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.converter,2.10.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.databinding.edit,1.4.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.databinding,1.4.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.ecore.edit,2.9.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.ecore.editor,2.10.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.ecore,2.10.1.v20140901-1043
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.edit,2.10.1.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.edit.ui,2.10.1.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf,2.10.1.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.mapping.ecore.editor,2.9.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.mapping.ecore,2.8.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.mapping,2.8.0.v20140901-1055
No repository found containing: org.eclipse.update.feature,org.eclipse.emf.mapping.ui,2.8.0.v20140901-1055
[/text]

 

 