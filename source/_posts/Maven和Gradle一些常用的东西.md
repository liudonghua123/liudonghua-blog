---
title: Maven和Gradle一些常用的东西
tags:
  - gradle
  - maven
id: 342
categories:
  - 学习
date: 2014-10-24 17:49:07
---

[toc]

###### **这里记录一些maven和gradle中可能会经常用到的东西...
Maven下载依赖项源码和javadoc**

<!--more-->

Maven中定义的依赖库默认是不下载源码和javadoc的，如果需要下载源码或者javadoc，可以使用以下几种方式
1\. 可以在命令行上参数方式
同时下载源码和javadoc

```shell
mvn depdency:resolve -DdownloadSources -DdownloadJavadocs
```

只下载源码

```shell
mvn dependency:sources
```

只下载javadoc

```shell
mvn dependency:resolve -Dclassifier=javadoc
```

2\. 在项目pom.xml文件中添加如下配置

```shell
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-eclipse-plugin</artifactId>
            <configuration>
                <downloadSources>true</downloadSources>
                <downloadJavadocs>true</downloadJavadocs>
            </configuration>
        </plugin>
    </plugins>
</build>
```

3\. 在更高层次全局设置(~/.m2/settings.xml).

```shell
<profiles>
    <profile>
        <id>downloadSources</id>
        <properties>
            <downloadSources>true</downloadSources>
            <downloadJavadocs>true</downloadJavadocs>
        </properties>
    </profile>
</profiles>

<activeProfiles>
    <activeProfile>downloadSources</activeProfile>
</activeProfiles>
```

4\. 在Eclipse的maven插件设置中设置，如下图所示
[![eclipse_maven_settings](/resources/2014/10/eclipse_maven_settings-293x300.png)](/resources/2014/10/eclipse_maven_settings.png)

###### **Maven创建web工程**

使用如下命令

```shell
mvn archetype:generate -DgroupId={project-packaging} -DartifactId={project-name} -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
```

进一步使用以下命令可以生成eclipse java web工程项目，并且下载依赖项，然后就可以在eclipse中导入已存在的普通eclipse项目。
或者也可以不执行以下命令，直接在eclipse中导入maven项目。

```shell
mvn eclipse:eclipse -Dwtpversion=2.0
```

mvn eclipse:eclipse 把maven项目转换成eclipse项目，"-Dwtpversion=2.0"是指定转换成eclipse web项目。

###### **Gradle中使用provided scope的depedency**

目前官方暂时不支持，见[Provide a 'provided' configuration](https://issues.gradle.org/browse/GRADLE-784)，不过可以通过以下方式，在gradle中添加

```shell
apply plugin: 'eclipse'  // Eclipse users only

configurations {
    provided
}

sourceSets {
    main.compileClasspath += configurations.provided
    test.compileClasspath += configurations.provided
    test.runtimeClasspath += configurations.provided
}

eclipse.classpath.plusConfigurations += configurations.provided  // Eclipse users only
```

第一行和最后一行仅针对eclipse适用

###### **Gradle下载依赖项源码和javadoc**

eclipse中通过eclipse插件下载

```shell
apply plugin: 'java'
apply plugin: 'eclipse'

eclipse {
    classpath {
       downloadSources=true
    }
}
```

intellij中通过idea插件下载

```shell
apply plugin: 'groovy'
apply plugin: 'idea'

repositories {
    mavenCentral()
    mavenRepo name: "Grails", url: "http://repo.grails.org/grails/repo/"
}

dependencies {
    groovy 'org.codehaus.groovy:groovy-all:2.0.4'
    compile 'org.slf4j:slf4j-log4j12:1.6.6', 'postgresql:postgresql:9.1-901.jdbc4', 'net.sourceforge.nekohtml:nekohtml:1.9.16'
    ['core', 'hibernate', 'plugin-datasource', 'plugin-domain-class'].each { plugin ->
        compile "org.grails:grails-$plugin:2.1.0"
    }
}

idea {
    module {
        downloadJavadoc = true
        downloadSources = true
    }
}

// Fat Jar Option (http://docs.codehaus.org/display/GRADLE/Cookbook#Cookbook-Creatingafatjar)
jar {
    from { configurations.compile.collect { it.isDirectory() ? it : zipTree(it) } }
}

task wrapper(type: Wrapper) {
    gradleVersion = '1.0'
}
```

###### **参考资料**

[maven-always-download-sources-and-javadocs](http://stackoverflow.com/questions/5780758/maven-always-download-sources-and-javadocs)
[get-source-jar-files-attached-to-eclipse-for-maven-managed-dependencies](http://stackoverflow.com/questions/310720/get-source-jar-files-attached-to-eclipse-for-maven-managed-dependencies)
[how-to-create-a-web-application-project-with-maven](http://www.mkyong.com/maven/how-to-create-a-web-application-project-with-maven/)
[how-to-use-provided-scope-for-jar-file-in-gradle-build](http://stackoverflow.com/questions/18738888/how-to-use-provided-scope-for-jar-file-in-gradle-build)
[provided-scope-in-gradle](http://www.sinking.in/blog/provided-scope-in-gradle/)
[emulating-mavens-provided-scope-in-gradle](http://blog.codeaholics.org/2012/emulating-mavens-provided-scope-in-gradle/)
[how-to-download-dependency-sources-for-gradle-project-in-idea](http://stackoverflow.com/questions/12718753/how-to-download-dependency-sources-for-gradle-project-in-idea)