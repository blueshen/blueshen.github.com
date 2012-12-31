---
layout: post
title: "从源码编译构建Jmeter"
date: 2012-12-31 15:48
comments: true
categories: jmeter
tags: [ jmeter, ant]
---
###获取Jmeter的源码###
Jmeter源码可以从SVN REPO找到，地址：<https://github.com/apache/jmeter>    
从GIT上也是可以的。地址：<https://github.com/apache/jmeter>    
Git clone:   

	git clone git://github.com/apache/jmeter.git jmeter
这样就把Jmeter的源码给放到了本地的jmeter文件夹内。

###配置并编译Jmeter###
<!--more-->
进入Jmeter目录，有个build.xml文件，显然是用ant管理的。   
此时，如果把项目import到eclipse内，会发现缺失很多依赖的jar包。此时不禁感叹，为啥不直接用maven管理呢，这样就不用管这些了。我第一次操作的时候害的我还去下载了一个最新的发行版本，然后从里面找各种依赖的jar包。后来发现，开发者已经为我们想到了这个。执行ant命令就可以直接下载：

	ant download_jars

一段时间的等待后，各种jar包就下载好了。要耐心等待，这个过程要下载很多包的。

接下来吗？直接执行：    

	ant install /ant clean install

提示BUILD SUCCESSFUL,这就表明已经编译成功了。


###运行Jmeter###
进入bin目录，执行jmeter.bat（windows）或者 ./jmeter（linux）,Jmeter的界面打开了，开始体验吧。