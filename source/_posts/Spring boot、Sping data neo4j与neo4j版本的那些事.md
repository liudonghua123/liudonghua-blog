---
title: Spring boot、Sping data neo4j与neo4j版本的那些事
tags:
  - neo4j
  - spring-boot
  - spring-data
  - spring-data-neo4j
id: 704
categories:
  - 学习
date: 2015-04-08 18:15:09
---

最近在写个小东西，用到了spring boot、spring data neo4j以及neo4j等一些组件，为使用最新版本的neo4j（2.2.0），在升级过程中遇到一些问题，在这里写点文字总结一下，方便过段时间忘了，可以回过头来看看！

<!--more-->

得力于spring boot很好的依赖继承性，我们在项目里可以仅添加一些主要、核心的库就可以了，比如如果我用spring+hibernate+mysql+...做一个web应用，之前可能会在pom.xml里定义一大推spring、hibernate、mysql、servlet、logback、...等等的依赖，但使用spring boot来做，只需要以下几个依赖，是不是世界一下子变清净了~（注意发挥此神奇功效的）
 
```xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.2.3.RELEASE</version>
	</parent>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
	</dependencies>
```

这时你看真正使用的依赖如下图居然有这么多，并且都还没显示完整。
![spring-boot-maven](/resources/2015/04/spring-boot-maven.png)

使用spring boot还有一个非常好的地方就是只需要配置一个parent的version，它会自动帮你配置合适（最优？）版本的相关库，让你专注于你自己的工作，不用再为各种库之间版本兼容问题烦恼！只要你代码写的好，升级的事不就是更新一下spring-boot-starter-parent一个版本吗~

好，回到正题，既然spring boot这么实用，那新项目必须得用上。

这段时间需要写一个使用spring框架配合neo4j的web应用，首先不加上web内容的依赖如下

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.2.2.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-neo4j</artifactId>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
        </dependency>
    </dependencies>
```

本地的neo4j是2.1.6的没问题，后来spring-boot-starter-parent升级到1.2.3.RELEASE也没问题，其实查看依赖库，1.2.2.RELEASE其实对应到neo4j的2.1.5，1.2.3.RELEASE对应到2.1.7，但前几天neo4j升级到了2.2.0(距离上个发布的2.1.6有接近半年时间了)，变化还是很大的，最明显的是browser模块有点“扁平化”了，界面上变漂亮了很多，而且默认进入需要验证了（可以通过修改neo4j-server.properties中的dbms.security.auth_enabled来关闭它），所以必须升级呀！

所以我就想让我的程序也可以用上最新的neo4j，去找找有没有新版本的spring-boot-starter-parent，发现即是最新的snapshot版本的也没有使用neo4j 2.2.0。没办法，那就在pom里先强制使用最新版的neo4j——在pom.xml里添加

```xml
		<dependency>
			<groupId>org.neo4j</groupId>
			<artifactId>neo4j</artifactId>
			<version>2.2.0</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-commons</artifactId>
			<version>1.10.0.RELEASE</version>
		</dependency>
```

添加后面一个spring-data-commons是为了覆盖spring-boot-starter-parent本省提供的较旧版本的spring-data-commons，neo4j-2.2.0需要新版本spring-data-commons的一些类。

以上的配置在非Web环境下就基本没问题了。但添加上web环境相关的东东，如spring-boot-starter-data-rest、spring-boot-starter-web时就会出现一些其他问题，但最终可以添加以下内容到dependencies最前面，防止特定的版本由于后面的一些依赖库覆盖掉。

```xml
		<dependency>
			<groupId>org.neo4j</groupId>
			<artifactId>neo4j</artifactId>
			<version>2.2.0</version>
		</dependency>
		<!-- Fix ClassNotFoundException: org.springframework.data.mapping.PersistentPropertyAccessor -->
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-commons</artifactId>
			<version>1.10.0.RELEASE</version>
		</dependency>
		<!-- Fix ClassNotFoundException: org.springframework.data.neo4j.core.UpdateableState -->
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-neo4j</artifactId>
			<version>3.3.0.RELEASE</version>
		</dependency>
		<!-- Fix ClassNotFoundException: org.neo4j.kernel.impl.nioneo.store.StoreId -->
		<dependency>
			<groupId>org.neo4j</groupId>
			<artifactId>neo4j-kernel</artifactId>
			<version>2.1.7</version>
		</dependency>
```

补充一点点，不是非常喜欢尝试新东西或追求极致稳定性的同学最好还是等spring boot升级，可能1.2.4.RELEASE或1.2.5.RELEASE就可以不用添加以上hack了。

除此之外再使用neo4j时还要注意是应用中内嵌neo4j还是使用外面的neo4j服务器。

我在我的代码里里是这样写的以便于可以根据需要动态的改变使用neo4j服务器的模式。

```java
	private static final String DATABASE_PATH = "resources/database_movie_recommend_neo4j.db";
	private static final String IS_EMBEDED = System.getenv("NEO4J_IS_EMBEDED") != null ? System.getenv("NEO4J_IS_EMBEDED") : "false";
	private static final String NEO4J_URL = System.getenv("NEO4J_URL") != null ? System.getenv("NEO4J_URL") : "http://localhost:7474/db/data/";

	@Bean
	public GraphDatabaseService graphDatabaseService() {
		if (Boolean.parseBoolean(IS_EMBEDED)) {
			return new GraphDatabaseFactory()
					.newEmbeddedDatabase(DATABASE_PATH);
		} else {
			return new SpringRestGraphDatabase(NEO4J_URL);
		}
	}
```

有空可以看看我做的这个小[Demo](http://liudonghua.com:1234)，目前使用的数据一个非常小的数据，就3个人5部电影，一个较大上千个人和上千个电影的数据正在建立人与人之前的联系呢！![](http://202.203.209.55:8080/wp-content/plugins/wp-emoji-one/icons/1F605.png)