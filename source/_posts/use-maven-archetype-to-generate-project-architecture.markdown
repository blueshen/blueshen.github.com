---
layout: post
title: "使用Maven Archetype来生成项目框架"
date: 2013-05-21 16:36
comments: true
categories: maven
tags: [ maven, archetype, generate, selenium ]
---
### Maven in 5 Minutes
[maven官方文档](http://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)的入门章节就介绍了如何创建一个maven项目。大致如下：

    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
我相信，很多人都是从这里开始的。但是为什么是这样呢？这里面都是怎么实现的？
其实，这里面是maven archetype的作用。它可以根据模板为你生成样例项目。
<!--more-->
### Maven Archetype Plugin
我们使用这个插件可以依据已经存在的项目来生成一个架构。这样其他人就可以使用这个架构来快速生成自己的项目了。
Maven Archetype Plugin有4个Goal:

- archetype:create 已过时，用于从archetype创建maven project
- archetype:generate  替代create,并且可以用交互的模式
- archetype:create-from-project 依据存在的project 创建archetype
- archetype:crawl 查找repo里存在的archetypes并更新目录。

### 实战演练一把
假设我们已经有了一个项目名叫seleniumframework-start:
进入到这个工程的根目录，首先执行：

    mvn clean archetype:create-from-project
这样就生成了一个archetype的代码。其路径位于./target/generated-sources/archetype目录内。其中pom.xml类似：

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>

      <groupId>com.baidu.selenium.archetypes</groupId>
      <artifactId>seleniumframework-start-archetype</artifactId>
      <version>1.0.0</version>
      <packaging>maven-archetype</packaging>

      <name>seleniumframework-start-archetype</name>

      <build>
        <extensions>
          <extension>
            <groupId>org.apache.maven.archetype</groupId>
            <artifactId>archetype-packaging</artifactId>
            <version>2.2</version>
          </extension>
        </extensions>

        <pluginManagement>
          <plugins>
            <plugin>
              <artifactId>maven-archetype-plugin</artifactId>
              <version>2.2</version>
            </plugin>
          </plugins>
        </pluginManagement>
      </build>
    </project>
这个就是要生成的seleniumframework-start-archetype的pom.xml了。一般情况下生成的代码是不能直接使用的，需要做一些修改。这其中主要有以下几个变量需要替换：

- ${groupId}
- ${artifactId}
- ${version}
- ${package}

找到./target/generated-sources/archetype/src/main/resources/archetype-resources目录，这里面就是要生成的文件模板了，这个时候需要注意以上几个占位符出现的地方是否正确，可以按照需要进行修改。常见的情况是把XML文件里的`<?xml version="1.0" encoding="UTF-8"?>`转为了`<?xml version="${version}" encoding="UTF-8"?>^`。这并不是我们想要的，`${version}`是用来指项目的版本号，因此此处去掉占位符。有的其他地方，应该要使用占位符的可以根据需要修改。修改完毕，那么返回./target/generated-sources/archetype目录，install之：

    mvn clean install
这样，这个archetype就安装到了local repository。
如何使用呢？咱们试试。

    mvn archetype:generate -DarchetypeGroupId=com.baidu.selenium.archetypes \
    -DarchetypeArtifactId=seleniumframework-start-archetype -DarchetypeVersion=1.0.0
其中，有一些交互信息需要确认。依次是groupId,artifactId,version,package。按照要求输入后，这些信息就是用来分别替代上面说的几个变量了，项目就顺利生成了。
此时，seleniumframework-start-archetype还只能自己使用，因为它只存在于local repository内。为了让大家共享成果，将这个包deploy到伺服器就OK了。


