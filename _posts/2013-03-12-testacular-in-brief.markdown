---
layout: post
title: "Karma  or Testacular 简介"
date: 2013-03-12 15:12
comments: true
categories: nodejs
tags: [ Testacular, JsTestDriver, nodejs, jenkins ]
---
##Karma/Testacular是什么？##
在2012年11月，Google开源了[Testacular](http://testacular.github.com/)，一个基于Node.js的JavaScript测试执行过程管理工具（Test Runner）。该工具可用于测试所有主流Web浏览器，也可集成到CI（Continuous integration）工具，也可和其他代码编辑器一起使用。   

Testacular可以在不同的桌面或移动设备浏览器上，或在持续集成的服务器上测试JavaScript代码。Testacular支持chrome、ChromeCanary、 Safari、Firefox、IE、Opera、PhantomJS，知道如何捕获浏览器正使用的默认路径，这些路径可能在启动器配置文件被忽视（overridden）。Testacular就是一个可以和多重测试框架协作的测试执行过程管理工具，它有针对Jasmine、Mocha和AngularJS的适配器，它也可以与[Jenkins](http://jenkins-ci.org/)或[Travis](https://travis-ci.org/)整合，用于执行持续集成测试。  
 
<!--more-->
这个测试工具的一个强大特性就是，它可以监控一套文件的变换，并立即开始测试已保存的文件，用户无需离开文本编辑器。测试结果通常显示在命令行中，而非代码编辑器。这也就让Testacular基本可以和任何JS编辑器一起使用。为更好结果，它可以整合到[WebStorm](http://www.jetbrains.com/webstorm/)中，而WebStorm持错误栈追踪和单元测试调试。  

为更好运行，Testacular需要Node.js和一个配置文件，该配置文件包括：待测试的文件、需忽略的文件、基本路径、web服务器端口、日子等级等。（配置文件样例）  

说到Testacular的性能，Google工程师Vojta Jína在Chrome Canary和Chrome做了一个演示，用WebStorm大约执行了1500个AngularJS测试，在5秒之内完成。

Jína也说到Testacular是受[JS Test Driver(JSTD)](http://code.google.com/p/js-test-driver/)的启发，但他们决定写一个完全不同的测试执行过程管理工具，因为JSTD有很多问题，他们想要一个能稳定并快速执行Javascript测试的工具。所以他们用了Socket.io库和Node.js。  

##Vojta Jína原版视频##
youtube(凸墙):   
<http://www.youtube.com/watch?v=5mHjJ4xf_K0>    
<http://www.youtube.com/watch?v=MVw8N3hTfCI>    
youku[个人转录]:    
<iframe height=498 width=510 src="http://player.youku.com/embed/XNTI2NTg0Nzky" frameborder=0 allowfullscreen></iframe>

<iframe height=498 width=510 src="http://player.youku.com/embed/XNTI2NTg0Mzc2" frameborder=0 allowfullscreen></iframe>
##Karma/Testacular 安装##
首先，保证已经有Node.js环境以及NPM。然后执行以下命令即可：   

	npm install -g karma/testacular
安装成功后，可以查看其支持的命令。   

	testacular --help
	Testacular - Spectacular Test Runner for JavaScript.

	Usage:
  	testacular <command>

	Commands:
  	start [<configFile>] [<options>] Start the server / do single run.
  	init [<configFile>] Initialize a config file.
  	run [<options>] Trigger a test run.

	Run --help with particular command to see its description and 	available options.

	Options:
  	--help     Print usage and options.
  	--version  Print current version.   
简单来看，就只有start,init,run这几个命令。start用于启动浏览器server,init用于辅助的生成配置文件，run用于驱动Test执行。  
下面就来看以下，最主要的部分，那就是配置文件了。

##Karma/Testacular配置文件##
这个配置文件，定义了Test执行所需要的各种选项，testacular正是通过这个文件来进行测试执行的。   
在GitHub上可以看到一个官方提供的默认样例<https://github.com/testacular/testacular/blob/master/test/client/testacular.conf.js>,可以看出里面有相当多的配置，还要里面都有一些注释的了，都大概能看懂一点。  
同样的，使用`karma/testacular init`命令也可以帮助你自动的生成一个配置文件。init后可以跟文件名，如果不写，默认的文件名就是karma/testacular.conf.js。对应的`karma/testacular start`也会默认搜索当前目录下的karma/testacular.conf.js来启动。    
下面，我们来生成一个看看：   

	karma/testacular init my.conf.js

	Which testing framework do you want to use ?
	Press tab to list possible options. Enter to move to the next 	question.
	> mocha

	Do you want to use Require.js ?
	This will add Require.js adapter into files.
	Press tab to list possible options. Enter to move to the next question.
	> no

	Do you want to capture a browser automatically ?
	Press tab to list possible options. Enter empty string to move to the next question.
	> Firefox
	......

	Config file generated at "/home/shenyanchao/tmp/my.conf.js".
这样就生成了一个my.conf.js文件。其中要我们自己要做的就是选择一下而已。需要注意的是，正如提示所说，选择切换使用的是**Tab**。  
此时，执行`testacular start my.conf.js`,可以发现，浏览器已经启动了。   
 
 
![Testacular启动](/images/blog/testacular-run-in-firefox.png)     


**配置文件参数：**  
	autoWatch

	类型: Boolean
	默认: false
	命令行: --auto-watch, --no-auto-watch
	详细介绍:当检测到文件内容变化的时候，是不是自动的重新运行Test

	basePath

	类型: String
	默认: ''
	详细介绍: 基本路径，用来解决相对路径问题。
 
	browsers

	类型: Array
	默认: []
	命令行: --browsers Chrome,Firefox
	取值:
	Chrome
	ChromeCanary
	Firefox
	Opera
	Safari
	PhantomJS
	IE
	详细介绍: 定义一组需要启动的浏览器，那么所有测试将分别在各个浏览器运行并给出结果。关闭的时候也同时全部关闭。

	captureTimeout

	类型: Number
	默认: 60000
	详细介绍: 捕获浏览器的超时时间 (单位 ms)。超时后，testacular会关闭然后重新尝试。  

	colors

	类型: Boolean
	默认: true
	命令行: --colors, --no-colors
	详细介绍: 在reporters和logs里面是否启用色彩。
	exclude

	类型: Array
	默认: []
	详细介绍: 排除在外的文件列表或者正则表达式

	files

	类型: Array
	默认: []
	详细介绍: 要加载的文件列表或者正则表达式

	hostname

	类型: String
	默认: 'localhost'
	详细介绍: 启动的浏览器主机名

	logLevel

	类型: Constant
	默认: LOG_INFO
	命令行: --log-level debug
	取值:
	LOG_DISABLE
	LOG_ERROR
	LOG_WARN
	LOG_INFO
	LOG_DEBUG
	详细介绍: 日志级别.

	loggers

	类型: Array
	默认: [{type: 'console'}]
	详细介绍: 定义日志目标。比如log4js

	port

	类型: Number
	默认: 9876
	命令行: --port 9876
	详细介绍: web服务的监听端口

	preprocessors

	类型: Object
	默认: {'**/*.coffee': 'coffee'}
	详细介绍: 前置处理器的MAP

	proxies

	类型: Object
	默认: {}
	详细介绍: 路径代理的映射MAP
	例如:
  	proxies =  {
    	'/static': 'http://gstatic.com',
    	'/web': 'http://localhost:9000'
  	};

	reportSlowerThan

	类型: Number
	默认: 0
	详细介绍: 这时一个以ms为单位的数值，如果test执行超过这个时间，那么Testacular会进行记录。 

	reporters

	类型: Array
	默认: ['progress']
	命令行: --reporters progress,growl
	取值:
	dots
	progress
	junit
	growl
	coverage
	详细介绍: 使用的报表列表

	runnerPort

	类型: Number
	默认: 9100
	命令行: --runner-port 9100
	详细介绍: 使用testacular run时，服务器的监听端口

	singleRun

	类型: Boolean
	默认: false
	命令行: --single-run, no-single-run
	详细介绍: CI模式。如为true，就会在所有浏览器运行，运行结束后关闭浏览器，返回码0，失败返回1.

	urlRoot

	类型: String
	默认: '/'
	详细介绍: 基本URL，相当于一个URL默认的前缀。尤其在使用proxies时有用。
##browser无法启动？##
当在karma/testacular.conf.js中配置完browsersCanary，有可能会出现无法启动浏览器的情况。testacular会在一套默认的路径下进行尝试加载启动浏览器，而在不同的操作系统下默认位置是不同的。
如果无法找到，可以通过覆盖`<BROWSER>_BIN`来解决。   
比如：   

	export CHROME_BIN=/usr/local/bin/my-chrome-build
	export CHROME_CANARY_BIN=/usr/local/bin/my-chrome-build
 	export PHANTOMJS_BIN=$HOME/local/bin/phantomjs
就是要设置相应的变量。在windows下自然就是添加相应的环境变量了。这样配置后，testacular就直到从哪儿加载启动浏览器了。   
##写在Testacular学习之后##
Testacular应该是Google[AngularJS](http://angularjs.org/)的副产品。出于CommonJS的规范，以及对产品质量的保证。AngularJS只身需要进行单元测试，而在测试过程中遇到了种种的问题。也许他们最开始就是使用JsTestDriver来驱动测试的，后来发现不能满足需求，或者能更好。因此Testacular出现了，并开源了出来。  
以上，存在一定的个人猜测，但是其产生的过程值得好好学习。   
##改名为Karma##
2013年3月18日，Testacular更名为Karma，版本从V0.6.0直接升为V0.8.0，并在GitHub上提交时评论为`chore: rename this shit to Karma`。具体什么原因，不得而知。也许是因为令人诟病的Testacular名字不好听吧。不过功能都是一样的，只是使用的时候，testacular变为karma了。所以上面文档中的操作，只需要全部替换即可。    