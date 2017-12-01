---
title: gradle wrapper缓存目录结构分析
tags:
  - bc
  - gradle
  - md5
id: 380
categories:
  - 学习
date: 2014-10-28 21:17:29
---

使用gradle wrapper虽然带来了方便，但首次使用会去下载相应版本的gradle，我最近经常遇到下载速度慢，下载完文件又损坏，非常痛苦，所以就稍微研究一下gradle wrapper缓存的结构。<!--more-->
其缓存位置是在~/.gradle/wrapper/dists 下，以下是我的目录层次结构

```shell
LiudonghuadeMacBook-Air:dists liudonghua$ pwd
/Users/liudonghua/.gradle/wrapper/dists
LiudonghuadeMacBook-Air:dists liudonghua$ tree -L 3
.
├── gradle-1.10-all
│   └── 6vpvhqu0efs1fqmqr2decq1v12
│       ├── gradle-1.10
│       ├── gradle-1.10-all.zip
│       ├── gradle-1.10-all.zip.lck
│       └── gradle-1.10-all.zip.ok
├── gradle-1.10-bin
│   └── 6oa4rff9viiqskhgd6uns5v1f8
│       ├── gradle-1.10
│       ├── gradle-1.10-bin.zip
│       ├── gradle-1.10-bin.zip.lck
│       └── gradle-1.10-bin.zip.ok
├── gradle-1.11-all
│   └── 7qd8qq8te5j4f5q9aaei3gh3lj
│       ├── gradle-1.11
│       ├── gradle-1.11-all.zip
│       ├── gradle-1.11-all.zip.lck
│       └── gradle-1.11-all.zip.ok
├── gradle-1.11-bin
│   └── 4h5v8877arc3jhuqbm3osbr7o7
│       ├── gradle-1.11
│       ├── gradle-1.11-bin.zip
│       ├── gradle-1.11-bin.zip.lck
│       └── gradle-1.11-bin.zip.ok
├── gradle-1.12-all
│   └── 2apkk7d25miauqf1pdjp1bm0uo
│       ├── gradle-1.12
│       ├── gradle-1.12-all.zip
│       ├── gradle-1.12-all.zip.lck
│       └── gradle-1.12-all.zip.ok
├── gradle-1.12-bin
│   └── 64p5p2nte80b6rt6bb068pabi6
│       ├── gradle-1.12
│       ├── gradle-1.12-bin.zip
│       ├── gradle-1.12-bin.zip.lck
│       └── gradle-1.12-bin.zip.ok
└── gradle-2.1-all
    └── 6n6ckv1svbjkm5ebj9is3php05
        ├── gradle-2.1
        ├── gradle-2.1-all.zip
        ├── gradle-2.1-all.zip.lck
        └── gradle-2.1-all.zip.ok

21 directories, 21 files
LiudonghuadeMacBook-Air:dists liudonghua$
```

其中gradle各版本名称下还有一层文件名，如6vpvhqu0efs1fqmqr2decq1v12、6oa4rff9viiqskhgd6uns5v1f8等，当时就猜想应该是文件内容的一些hash值，但我观察到下载gradle时就已经下载创建了这层文件夹，并且官方下载站点也没有相应文件的md5或sha，很有可能是文件名或gradle-wrapper.properties中定义的distributionUrl的hash，自己也尝试hash对比但都不匹配，并且在网上也查了很多资料都没有找到这方面的信息，所以就打算自己研究一下，看一下代码里面是怎么实现的。

一般本地如果没有相应的gradle wrapper，那执行./gradlew assemble是就会自动下载，所以从gradlew这个脚本文件找起，从这个脚本文件中最后

```shell
exec "$JAVACMD" "${JVM_OPTS[@]}" -classpath "$CLASSPATH" org.gradle.wrapper.GradleWrapperMain "$@"
```

或Windows版的gradlew.bat

```shell
"%JAVA_EXE%" %DEFAULT_JVM_OPTS% %JAVA_OPTS% %GRADLE_OPTS% "-Dorg.gradle.appname=%APP_BASE_NAME%" -classpath "%CLASSPATH%" org.gradle.wrapper.GradleWrapperMain %CMD_LINE_ARGS%
```

中可以看出是执行了org.gradle.wrapper.GradleWrapperMain，并且把命令行参数带进去的，所以后面查看gradle源码的[GradleWrapperMain](https://github.com/gradle/gradle/blob/REL_2.1/subprojects/wrapper/src/main/java/org/gradle/wrapper/GradleWrapperMain.java)，位于subprojects/wrapper/src/main/java/org/gradle/wrapper/GradleWrapperMain.java，在wrapper的子项目中，分析里面的main方法，注意到下面的高亮显示部分

```java
    public static void main(String[] args) throws Exception {
        File wrapperJar = wrapperJar();
        File propertiesFile = wrapperProperties(wrapperJar);
        File rootDir = rootDir(wrapperJar);

        CommandLineParser parser = new CommandLineParser();
        parser.allowUnknownOptions();
        parser.option(GRADLE_USER_HOME_OPTION, GRADLE_USER_HOME_DETAILED_OPTION).hasArgument();

        SystemPropertiesCommandLineConverter converter = new SystemPropertiesCommandLineConverter();
        converter.configure(parser);

        ParsedCommandLine options = parser.parse(args);

        Properties systemProperties = System.getProperties();
        systemProperties.putAll(converter.convert(options, new HashMap<String, String>()));

        File gradleUserHome = gradleUserHome(options);

        addSystemProperties(gradleUserHome, rootDir);

        WrapperExecutor wrapperExecutor = WrapperExecutor.forWrapperPropertiesFile(propertiesFile, System.out);
        wrapperExecutor.execute(
                args,
                new Install(new Download("gradlew", wrapperVersion()), new PathAssembler(gradleUserHome)),
                new BootstrapMainStarter());
    }
```

然后顺着[Install](https://github.com/gradle/gradle/blob/REL_2.1/subprojects/wrapper/src/main/java/org/gradle/wrapper/Install.java)查找

```java
    public File createDist(WrapperConfiguration configuration) throws Exception {
        final URI distributionUrl = configuration.getDistribution();

        final PathAssembler.LocalDistribution localDistribution = pathAssembler.getDistribution(configuration);
        final File distDir = localDistribution.getDistributionDir();
        final File localZipFile = localDistribution.getZipFile();
```

找到[PathAssembler](https://github.com/gradle/gradle/blob/REL_2.1/subprojects/wrapper/src/main/java/org/gradle/wrapper/PathAssembler.java)

```java
    /**
     * Determines the local locations for the distribution to use given the supplied configuration.
     */
    public LocalDistribution getDistribution(WrapperConfiguration configuration) {
        String baseName = getDistName(configuration.getDistribution());
        String distName = removeExtension(baseName);
        String rootDirName = rootDirName(distName, configuration);
        File distDir = new File(getBaseDir(configuration.getDistributionBase()), configuration.getDistributionPath() + "/" + rootDirName);
        File distZip = new File(getBaseDir(configuration.getZipBase()), configuration.getZipPath() + "/" + rootDirName + "/" + baseName);
        return new LocalDistribution(distDir, distZip);
    }

    private String rootDirName(String distName, WrapperConfiguration configuration) {
        String urlHash = getHash(configuration.getDistribution().toString());
        return String.format("%s/%s", distName, urlHash);
    }

    /**
     * This method computes a hash of the provided {@code string}.
     * <p>
     * The algorithm in use by this method is as follows:
     * <ol>
     *    <li>Compute the MD5 value of {@code string}.</li>
     *    <li>Truncate leading zeros (i.e., treat the MD5 value as a number).</li>
     *    <li>Convert to base 36 (the characters {@code 0-9a-z}).</li>
     * </ol>
     */
    private String getHash(String string) {
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("MD5");
            byte[] bytes = string.getBytes();
            messageDigest.update(bytes);
            return new BigInteger(1, messageDigest.digest()).toString(36);
        } catch (Exception e) {
            throw new RuntimeException("Could not hash input string.", e);
        }
    }
```

应该就是上面的第十四行了，我自己也尝试调用getHash方法验证，但始终得不到预期的结果，想了很久，突然想到虽然都是md5（128bit），但我们平时使用的都是16进制的字符表示，有128/4=32位，但这里如"6n6ckv1svbjkm5ebj9is3php05"只有26位，代码中是使用36进制的字符表示，我尝试改成32位的试试，没想到突然得到了预期的正确结果，但代码里明明是使用36进制，而不是32进制，是不是最近才修改了代码，所以又看一下之前tag的代码，如1.12版本的果然是32而不是36（可以在[这里](https://github.com/gradle/gradle/commit/2e6659547e71bb3fca1c952d823ec660433ab5d9)看到这次修改相应的commit，至少在2.1正式版中还是32进制表示的）
附上一些测试的shell命令

```shell
LiudonghuadeMacBook-Air:dists liudonghua$ md5 -s "http://services.gradle.org/distributions/gradle-2.1-all.zip"
MD5 ("http://services.gradle.org/distributions/gradle-2.1-all.zip") = d73329f0f3eb9d2c572e69970798e405
LiudonghuadeMacBook-Air:dists liudonghua$
LiudonghuadeMacBook-Air:dists liudonghua$ echo d73329f0f3eb9d2c572e69970798e405 |  awk '{print toupper($0)'}
D73329F0F3EB9D2C572E69970798E405
LiudonghuadeMacBook-Air:dists liudonghua$ echo "obase=32;ibase=16; $(echo $(echo $(md5 -q -s 'http://services.gradle.org/distributions/gradle-2.1-all.zip') | awk '{print toupper($0)}'))" | bc
 06 23 06 12 20 31 01 28 31 11 19 20 22 05 14 11 19 09 18 28 03 25 1\
7 25 00 05
LiudonghuadeMacBook-Air:dists liudonghua$ bc
bc 1.06
Copyright 1991-1994, 1997, 1998, 2000 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
obase=32
ibase=16
D73329F0F3EB9D2C572E69970798E405
 06 23 06 12 20 31 01 28 31 11 19 20 22 05 14 11 19 09 18 28 03 25 1\
7 25 00 05
quit
LiudonghuadeMacBook-Air:dists liudonghua$
```

注意bc做进制转换时，一定要先设置obase，在设置inbase，并且数据中的字母要大写

现在知道了原理，但后面如果遇到一些版本下载有问题，就直接创建指定的目录，然后把用其他方式下载回来的zip放到指定目录中，就会自动解压安装了。如下载gradle-1.12-all有问题，那就创建~/.gradle/wrapper/dists/gradle-1.12-all/2apkk7d25miauqf1pdjp1bm0uo，这里2apkk7d25miauqf1pdjp1bm0uo是字符串"http://services.gradle.org/distributions/gradle-1.12-all.zip"的32位md5表示形式（在未来版本中这会变成36位），再把gradle-1.12-all.zip直接放在里面就可以了。

注：补充gradle依赖库缓存位置(~/.gradle/caches/modules-2/files-2.1)及目录结构

```shell
LiudonghuadeMacBook-Air:files-2.1 liudonghua$ pwd
/Users/liudonghua/.gradle/caches/modules-2/files-2.1
LiudonghuadeMacBook-Air:files-2.1 liudonghua$ ll
total 0
drwxr-xr-x@ 39 liudonghua  staff  1326 Oct 18 19:32 .
drwxr-xr-x@  8 liudonghua  staff   272 Oct 28 19:58 ..
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 aopalliance
drwxr-xr-x@  4 liudonghua  staff   136 Oct 18 19:32 com.actionbarsherlock
drwxr-xr-x@  6 liudonghua  staff   204 Oct 18 19:32 com.android.tools
drwxr-xr-x@  7 liudonghua  staff   238 Oct 18 19:32 com.android.tools.build
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 com.android.tools.ddms
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 com.android.tools.external.lombok
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 com.android.tools.layoutlib
drwxr-xr-x@  5 liudonghua  staff   170 Oct 18 19:32 com.android.tools.lint
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 com.github.dcendents
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 com.google.android
drwxr-xr-x@  4 liudonghua  staff   136 Oct 18 19:32 com.google.guava
drwxr-xr-x@  5 liudonghua  staff   170 Oct 18 19:32 com.googlecode.androidannotations
drwxr-xr-x@  4 liudonghua  staff   136 Oct 18 19:32 com.joanzapata.pdfview
drwxr-xr-x@  4 liudonghua  staff   136 Oct 18 19:32 com.nineoldandroids
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 com.squareup
drwxr-xr-x@  4 liudonghua  staff   136 Oct 18 19:32 com.sun.codemodel
drwxr-xr-x@  4 liudonghua  staff   136 Oct 18 19:32 com.twotoasters.jazzylistview
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 com.viewpagerindicator
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 commons-codec
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 commons-logging
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 de.keyboardsurfer.android.widget
drwxr-xr-x@  4 liudonghua  staff   136 Oct 18 19:32 javax.servlet
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 joda-time
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 kxml2
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 net.java
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 net.sf.kxml
drwxr-xr-x@  5 liudonghua  staff   170 Oct 18 19:32 net.sf.proguard
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 org.apache
drwxr-xr-x@  4 liudonghua  staff   136 Oct 18 19:32 org.apache.commons
drwxr-xr-x@  8 liudonghua  staff   272 Oct 18 19:32 org.apache.httpcomponents
drwxr-xr-x@  4 liudonghua  staff   136 Oct 18 19:32 org.bouncycastle
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 org.eclipse.jdt.core.compiler
drwxr-xr-x@  7 liudonghua  staff   238 Oct 18 19:32 org.jacoco
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 org.ow2
drwxr-xr-x@  7 liudonghua  staff   238 Oct 18 19:32 org.ow2.asm
drwxr-xr-x@  3 liudonghua  staff   102 Oct 18 19:32 org.sonatype.oss
drwxr-xr-x@  9 liudonghua  staff   306 Oct 18 19:32 org.springframework
LiudonghuadeMacBook-Air:files-2.1 liudonghua$
LiudonghuadeMacBook-Air:files-2.1 liudonghua$ cd javax.servlet/
jstl/        servlet-api/
LiudonghuadeMacBook-Air:files-2.1 liudonghua$ cd javax.servlet/servlet-api/2.5/
21599814ad9a605b86f3e6381571beccd861a32/  a159fa05cce714c83deff647655dd53db064b21c/
5959582d97d8b61f4d154ca9e495aafd16726e34/
LiudonghuadeMacBook-Air:files-2.1 liudonghua$ cd javax.servlet/servlet-api/2.5/
LiudonghuadeMacBook-Air:2.5 liudonghua$
LiudonghuadeMacBook-Air:2.5 liudonghua$ tree
.
├── 21599814ad9a605b86f3e6381571beccd861a32
│   └── servlet-api-2.5-sources.jar
├── 5959582d97d8b61f4d154ca9e495aafd16726e34
│   └── servlet-api-2.5.jar
└── a159fa05cce714c83deff647655dd53db064b21c
    └── servlet-api-2.5.pom

3 directories, 3 files
LiudonghuadeMacBook-Air:2.5 liudonghua$
LiudonghuadeMacBook-Air:2.5 liudonghua$ shasum 21599814ad9a605b86f3e6381571beccd861a32/servlet-api-2.5-sources.jar
021599814ad9a605b86f3e6381571beccd861a32  21599814ad9a605b86f3e6381571beccd861a32/servlet-api-2.5-sources.jar
LiudonghuadeMacBook-Air:2.5 liudonghua$
LiudonghuadeMacBook-Air:2.5 liudonghua$ shasum 5959582d97d8b61f4d154ca9e495aafd16726e34/servlet-api-2.5.jar
5959582d97d8b61f4d154ca9e495aafd16726e34  5959582d97d8b61f4d154ca9e495aafd16726e34/servlet-api-2.5.jar
LiudonghuadeMacBook-Air:2.5 liudonghua$
```

这与maven的管理方式还是有些不同，目录层次减少了，并且也加入了强制文件验证。