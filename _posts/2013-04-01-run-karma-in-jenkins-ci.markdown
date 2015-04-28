---
layout: post
title: "Karma与Jenkins-CI集成"
date: 2013-04-01 19:38
comments: true
categories: nodejs
tags: [ karma, testacular, jenkins, junit, coverage, cobertura ]
---
Jenkins是一款目前最为流行的持续集成工具，那么，如何让Karma的能集成到Jenkins，并自动执行呢？

###前提条件###
Jenkins Server上（可以是Master，也许是Slave结点，总之在那个Server上跑，就需要安装），安装：   

* Node
* Karma

###配置Karma.conf.js文件###
必须保证：

	singleRun = true;
只有这样，才能保证运行Test后，浏览器自动退出，不影响下次执行。    
在Jenkins中，也许你想查看测试结果，这个时候可以借助junit reporter。

	reporters = ['junit'];
	junitReporter = {
  		outputFile: 'test-results.xml'
	};
那么，Junit格式的测试结果就存到了test-results.xml中。    

另外一种情况，我可能还想查看一下代码覆盖率。Karma也是支持的，要进行以下的配置：   

	reporters = ['coverage'];

	preprocessors = {
    	'src/*.js': 'coverage'
	};

	coverageReporter = {
    	type : 'cobertura',
    	dir : 'coverage/'
	};
这里，reporters指出了要生成coverage报告。preprocessors指明了要统计覆盖率的源码。coverageReporter里，指明type为cobertura，dir则是报告路径。type用多种选择，其中cobertura为Jenkins专属的。   
<!--more-->
###一个Jenkins Job###
1.新建一个自由风格（freestyle）的Job即可。   
2.Restrict where this project can be run 里面填好前提条件中的机器名。当然如果直接是在Master结点，这个可以忽略。   
3.源码管理部分，填写repository url。    
4.在构建里，直接填写`karma start`即可。   
5.在构建后操作里。选择Publish Cobertura Coverage Report,并指定生成的XML地址。如下图：   
   
![karma-jenkins-cobertura](/images/blog/karma-jenkins-cobertura.png)

6.在构建后操作里。选择Publish JUnit test result report,同样指定report的XML路径。如图：   
  
![karma-jenkins-junit](/images/blog/karma-jenkins-junit.png)  

###Jenkins运行结果###
Code Coverage结果：   
![](/images/blog/karma-jenkins-codecoverage.png)

点击进入后，可以看到具体的覆盖率情况：   
![](/images/blog/karma-jenkins-codecoverage-detail.png)

JUnit结果：   
![](/images/blog/karma-jenkins-junit-report.png)  

点进去后可以查看详细信息：  
![](/images/blog/karma-jenkins-junit-report-detail.png)

###关于Coverage###
coverageReporter的类型有以下几种：   

- html (default)
- lcov (lcov and html)
- lcovonly
- text (standard output)
- text-summary (standard output)
- cobertura (xml format supported by Jenkins)   

Karma是使用[istanbul](http://gotwarlost.github.com/istanbul/)来生成报告的，上面在Jenkins种使用的cobertura类型。如果不在CI环境中，那么可以考虑使用lcon或者html类型，report也是相当好看呢。   
以下是lcov类型的Coverage结果： 

![](/images/blog/karma-lcov-1.png)

而下面的结果，则指明了哪行源码被覆盖了。   
![](/images/blog/karma-lcov-2.png)
