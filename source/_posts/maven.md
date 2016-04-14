title: 通过maven管理项目
date: 2016-01-24 13:37:56
tags: ["maven"]
---
最初我自己搭建开发环境的时候，整理与下载依赖非常头疼。花时间整理和下载非常麻烦，后来发现maven后便一直使用maven下载依赖包了。
maven是一个项目管理工具,它规范了整个项目的构建，使用maven能更方便高效的管理Java项目与多人协作。
<h2 id="setup">安装maven</h2>
>[官方地址](http://maven.apache.org/download.cgi)

下载对应的maven版本（当然你要先安装Java），解压到对应目录，配置环境变量M2_HOME到对应目录，将M2_HOME的bin目录加入PATH中。

运行命令`mvn -v`检查安装是否成功

<h2 id="info">基本概念</h2>
+ 一个maven项目的核心是pom.xml，它定义了项目的基本信息，描述了项目构建与依赖等
+ 一个maven项目的坐标由groupId,artifactId和version确定
+ maven有远程仓库与本地仓库，依赖寻找会先在本地仓库寻找再按配置的远程仓库寻找
+ maven项目之间依赖是通过jar包完成的，就算是本地的两个项目，也是会在本地仓库寻找
+ maven的标准目录结构

>[官方地址](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)

src
　　\-main
　　　　\-bin
　　　　\-java 源代码
　　　　\-resources 资源目录
　　　　\-filters Resource filter files
　　　　\-assembly 组件的描述配置（如何打包）
　　　　\-config 配置文件目录
　　　　\-webapp web应用目录
　　\-test
　　　　\–java 单元测试目录
　　　　\–resources 测试资源目录
　　　　\–filters
target 构建目录

<h2 id="command">常用命令</h2>
>[官方地址](https://maven.apache.org/guides/getting-started/index.html)

+ mvn archetype:create 创建 Maven 项目
+ mvn compile 编译源代码
+ mvn test-compile 编译测试代码
+ mvn test 运行单元测试
+ mvn site 生成项目相关信息的网站
+ mvn clean 清空构建目录
+ mvn package 打包项目
+ mvn install 编译打包到本地仓库
+ mvn deploy 将项目上传至远程仓库
+ mvn eclipse:eclipse 生成 Eclipse 项目文件

<h2 id="dependency">坐标与依赖</h2>
> 坐标是用于确定一个项目的唯一标识，它由三部分组成groupId，artifactId和version [官方地址](https://maven.apache.org/guides/mini/guide-naming-conventions.html)

groupId：当前maven项目对应的实际项目（可以认为是一个域下的项目组，如org.springframework）
artifactId：当前maven项目或模块的Id
version：当前项目或模块版本

> maven依赖管理解决了最头疼的问题，多数情况下我们只用在pom.xml里配置maven就能自动解决，更多依赖机制参照 [官方地址](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

常用配置
grouId、artifactId、version：依赖的坐标（必需）
type：依赖的类型，对应的是打包方式packaging，多数情况下不需要配置（默认是jar）
scope：依赖范围
optional：依赖是否可选
exclusions：用来排除传递性依赖

多数情况下，依赖只需要配置坐标，如下配置了mybatis的依赖
``` +xml
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>3.3.1</version>
</dependency>
```

<h2 id="remote">配置远程仓库</h2>
>maven的默认远程仓库是[中央仓库](http://search.maven.org/)，国内使用很容易卡，以下配置将远程仓库改为[开源中国maven库](http://maven.oschina.net/)

maven的个人配置文件在个人目录（home或者是User下）的.m2/settings.xml
在mirrors节点中增加以下配置
``` +xml
<mirror>
    <id>nexus-osc</id>
    <mirrorOf>*,!nexus-eheres</mirrorOf>
    <name>Nexus osc</name>
    <url>http://maven.oschina.net/content/groups/public/</url>
</mirror>
```

<h2 id="private">搭建私有仓库</h2>
>多人开发maven项目，就需要搭建一个私有仓库来协作开发了（开发的jar包不能上传到公有仓库），这里使用[Nexus](http://www.sonatype.org/nexus/)来搭建私有仓库

1.下载nexus
`wget https://download.sonatype.com/downloads/products/nexus/nexus-professional-bundle-latest.tar.gz`

2.解压文件
`tar zxvf nexus-latest-bundle.tar.gz`

3.启动服务(不要使用root权限用户)
`cd /nexus/nexus-version/bin`
`./nexus start`

等待启动完成  就可以在http://localhost:8081/nexus 访问页面了
管理员的帐号默认为admin 密码admin123
![nexus界面](/img/nexus/nexus.png)

4.配置私有仓库
打开远程仓库索引下载
在View/Repositories -> Repositories 中 设置central的仓库configuration选项卡，将其中的Download Remote Indexes，把fasle改为true，保存并点击右键，选择update index
![设置索引下载](/img/nexus/remote.png)

在servers节点增加
``` +xml
<server>  
    <id>nexus-central</id>  
    <username>admin</username>  
    <password>admin123</password>  
</server>
<server>  
    <id>nexus-snapshots</id>  
    <username>admin</username>  
    <password>admin123</password>  
</server>
```
在mirrors节点增加
``` +xml
<mirror>
    <id>nexus-central</id>
    <mirrorOf>nexus-central</mirrorOf>
    <name>Nexus Snapshot Repository</name>
    <url>http://localhost:8081/nexus/content/repositories/central</url>
</mirror>
<mirror>
    <id>nexus-snapshots</id>
    <mirrorOf>nexus-snapshots</mirrorOf>
    <name>Nexus Snapshot Repository</name>
    <url>http://localhost:8081/nexus/content/repositories/snapshots</url>
</mirror>
```
