---
layout: post
title: "Sonar的安装与使用"
date: 2013-09-04 17:02
comments: true
categories: 测试
tags: [ sonar, pmd, checkstyle, findbugs, maven ]
---
### 什么是Sonar？
[Sonar](http://www.sonarqube.org/)是一个开源的代码质量管理平台。它能对代码进行如下7个维度的管理。

![sonar](http://www.sonarqube.org/wp-content/themes/sonar/images/7axes.png)
使用插件，它可以对20多种语言进行代码质量管理，这其中包括Java，C#，C/C++,PL/SQL等等。

### 安装Sonar
1.下载sonar,地址<http://www.sonarqube.org/downloads/>。通常选取稳定版本下载即可，这是一个zip文件。
2.解压下载的sonar到一个目录。我们称这个解压后的路径为SONAR_HOME
3.进入$SONAR_HOME/bin/${os-version}/,找到sonar.sh,执行`./sonar.sh console`即可。在windows下是StartSonar.bat。
4.现在进入<http://localhost:9000>,就看到了界面。默认的登录使用admin:admin

这个时候，Sonar已经运行啦。但是在生产环境是不行的。上面跑起来的只是一个样例，使用的是h2内存数据库。我们可不想重启服务后，生产环境的数据都没了。
<!--more-->

### 配置Sonar数据库
1.首先新建一个数据库。

```mysql
CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;
grant all privileges on sonar.* to 'sonar'@'%' identified by '你的密码';
flush privileges;
```
这样就准备好了数据库sonar,并授权给sonar这个用户。

2.找到$SONAR_HOME/conf/sonar.properties。
注释掉默认的数据库配置，然后配上自己的数据库信息即可。这里以mysql为例。

```shell
# Comment the following line to deactivate the default embedded database.
#sonar.jdbc.url:                            jdbc:h2:tcp://localhost:9092/sonar
#sonar.jdbc.driverClassName:                org.h2.Driver
-------------------
# The schema must be created first.
sonar.jdbc.username:                       sonar
sonar.jdbc.password:                       sonar
#----- MySQL 5.x
# Comment the embedded database and uncomment the following line to use MySQL
sonar.jdbc.url: jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true

# Optional properties
sonar.jdbc.driverClassName: com.mysql.jdbc.Driver
```
配置好之后，这样所有的数据都会存放到mysql内啦。不用再担心数据问题啦。要添加其他数据库，同理。

### 把Sonar变为中文
英文看这不方便啊。有2种方法可以将Sonar变为中文界面。

1.用管理员登录后，在Update Center种找到Localization里的Chinese Pack安装就可以了。
2.直接下载<http://repository.codehaus.org/org/codehaus/sonar-plugins/l10n/sonar-l10n-zh-plugin/1.6/sonar-l10n-zh-plugin-1.6.jar>这个插件jar包到$SONAR_HOME/extensions/plugins内，重启即可。

### 把Sonar放到JEE容器内
默认的情况下，sonar启动是采用内置的jetty的，为了方便管理，一般在生产环境可以放到JEE容器内，这里就以Tomcat为例了。
Sonar在经过上面几步的配置后，已经满足了基本的需求。接下来就可以进入到$SONAR_HOME/war/内。执行build-war命令。这样就生成了一个sonar.war，把这个war包发布到Tomcat即可。

### 如何对源码进行检测
1.配置maven的settings.xml,添加一下内容：

```xml
    <profile>
         <id>sonar</id>
         <activation>
             <activeByDefault>true</activeByDefault>
         </activation>
         <properties>
              <sonar.jdbc.url>
              jdbc:mysql://localhost:3306/sonar?useUnicode=true&amp;characterEncoding=utf8
              </sonar.jdbc.url>
              <sonar.jdbc.driver>com.mysql.jdbc.Driver</sonar.jdbc.driver>
              <sonar.jdbc.username>sonar</sonar.jdbc.username>
              <sonar.jdbc.password>sonar</sonar.jdbc.password>
             <sonar.host.url>http://localhost:9000/sonar</sonar.host.url>
         </properties>
      </profile>
```
其中的数据库配置以及sonar主机地址都依据实际进行修改即可。
2.在maven项目种执行

```shell
mvn clean install
mvn sonar:sonar
```
3.打开sonar主页，就可以看到结果了。

### Sonar与Jenkins的集成。

1.安装[jenkins-sonar-plugin](http://docs.codehaus.org/display/SONAR/Jenkins+Plugin)到Jenkins内。
2.在Jenkins里的系统配置中，填写Sonar安装信息。
3.在Jenkins的JOB中，配置post-build action中添加上Sonar即可。这样在项目构建后，会自动的执行Sonar分析。并将结果放在首页进行展现。

