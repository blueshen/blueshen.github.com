---
layout: post
title: "安装配置Maven"
date: 2012-10-19 23:40
comments: true
categories: maven
tags: [ maven, install, config ]
---
## Maven安装	
1.下载Maven：从[Maven官方](http://maven.apache.org/download.html)下载,并解压。比如解压到：D:\apache-maven-3.0.4     
2.配置环境变量：  

*  M2_HOME：D:\apache-maven-3.0.4
*  path：%M2_HOME%\bin;   

打开cmd命令窗口,验证配置是否成功：`mvn -v`。成功后会显示正确的版本信息。  
3.修改默认的repository   
maven默认放在C:\Users\user\.m2\repository目录下（不同系统可能不一样）。Window下一般不建议放在C盘。可以通过如下方式修改：   
进入%M2_HOME%\conf\settings.xml中，添加`<localRepository>e:/repo</localRepository>`这样一行。默认的路径就指向新的目录了。