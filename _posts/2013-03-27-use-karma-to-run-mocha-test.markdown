---
layout: post
title: "使用Karma来驱动mocha测试"
date: 2013-03-27 19:00
comments: true
categories: nodejs
tags: [ karma, testacular, mocha, idea ]
---
##从Testacular到Karma的变化##
2013年03月18日，Testacular正式被重命名为Karma。具体原因，讲起来缺也很滑稽。这里面不含有任何的商业成分，只是因为Testacular与Testicular很相似，因此令人感觉尴尬。仅仅此而已，谁让JsTestDriver已经被别人给拿走了。  
安装：  
	npm install -g karma
##什么时候使用Karma？##

* 在真实浏览器里测试。
* 在多种浏览器里进行测试（包括桌面、移动）。
* 在本地开发环境执行测试。
* 想在持续集成CI内运行测试。
* 想在每次保存代码时，自动执行测试。
* 热衷于terminal小黑屏。
* 不想陷入令人厌烦的测试生活。
* 想使用Istanbul自动生成coverage报告。
* 想在源码中使用RequireJS。

##Karma不是Testing Framework##
Karma自从出现，就是一直作为一个Test Runner而存在的，只是用来驱动测试的框架。不过到目前为止，它支持以下流行的测试框架。  

* Mocha
* Jasmine
* QUnit
<!--more-->
##Karma与Test Framework集成##
Karma对各种Test Framework的支持是以插件的模式进行支持的。  
只需要在karma.conf.js进行以下配置（mocha为例）：  

    frameworks = ['mocha'];

    plugins = [
    'karma-mocha'
    ];
在此处只是配置了一下，具体支持的plugin要在当前目录下进行安装：    

	npm install karma-mocha
其他框架依此类推。

##Karma报告##
Karma的报告（reporter）也是以插件模式进行的。 
####JUnit Reporter####
首先，要定义reporter类型，在karma.conf.js添加：  

	reporters = ['junit'];
如果想更近一步的话，可以配置一下报告的位置。   

	junitReporter = {
    	outputFile: 'junit-report/test-results.xml'
	};
报告配置完了，自然要有依赖啊。执行`npm install karma-junit-reporter`来安装。然后在加上这个plugin:   

	plugins = [
    	'karma-junit-reporter'
	];

####Coverage Reporter####
同JUnit Reporter一样，首先添加：  

	reporters = ['coverage'];
进一步的配置coverage report:   

	coverageReporter = {
    	type : 'cobertura',
    	dir : 'coverage/'
	};
其中，type用于指出报告类型；dir用于指出生成报告的存放目录。  
type包括：  

- html (default)
- lcov (lcov and html)
- lcovonly
- text (standard output)
- text-summary (standard output)
- cobertura (xml format supported by Jenkins)

下面，需要安装依赖`npm install karma-coverage`。并在配置文件内添加：  

	plugins = [
    	'karma-coverage'
	];


##创建一个样例##
1.执行‘karma init’,然后根据提示按Tab键进行相关选择。先生成一个默认的配置文件，这个是可以再修改的。    

2.创建一个src文件夹，用于存放待测试的JS代码。然后在创建一个test文件夹，用于存放自己写的单元测试代码。

3.以mocha为例，将mocha集成到karma中，使用karma来驱动测试。这需要在karma.conf.js里进行如下配置：  

	// Karma configuration
	// Generated on Tue Mar 19 2013 20:46:08 GMT+0800 (CST)

	// base path, that will be used to resolve files and exclude
	basePath = './';

	frameworks = ['mocha'];

	// list of files / patterns to load in the browser
	files = [
    	{pattern: 'node_modules/chai/chai.js',include: true},
    	'src/*.js',
    	'test/*.js'
	];


	// list of files to exclude
	exclude = [
    	'karma.conf.js'
	];


	// use dots reporter, as travis terminal does not support escaping sequences
	// possible values: 'dots', 'progress', 'junit', 'teamcity'
	// CLI --reporters progress
	reporters = ['progress','junit','coverage'];

	junitReporter = {
	    // will be resolved to basePath (in the same way as files/exclude patterns)
	    outputFile: 'junit-report/test-results.xml'
	};

	preprocessors = {
	    'src/*.js': 'coverage'
	};

	//Code Coverage options. report type available:
	//- html (default)
	//- lcov (lcov and html)
	//- lcovonly
	//- text (standard output)
	//- text-summary (standard output)
	//- cobertura (xml format supported by Jenkins)
	coverageReporter = {
	    // cf. http://gotwarlost.github.com/istanbul/public/apidocs/
	    type : 'cobertura',
	    dir : 'coverage/'
	};


	// web server port
	port = 9876;


	// cli runner port
	runnerPort = 9100;


	// enable / disable colors in the output (reporters and logs)
	colors = true;


	// level of logging
	// possible values: LOG_DISABLE || LOG_ERROR || LOG_WARN || LOG_INFO || LOG_DEBUG
	logLevel = LOG_DEBUG;


	// enable / disable watching file and executing tests whenever any file changes
	autoWatch = true;


	// Start these browsers, currently available:
	// - Chrome
	// - ChromeCanary
	// - Firefox
	// - Opera
	// - Safari (only Mac)
	// - PhantomJS
	// - IE (only Windows)
	// CLI --browsers Chrome,Firefox,Safari
	browsers = ['Chrome'];


	// If browser does not capture in given timeout [ms], kill it
	captureTimeout = 6000;


	// Continuous Integration mode
	// if true, it capture browsers, run tests and exit
	singleRun = true;


	plugins = [
	    'karma-mocha',
	    'karma-chrome-launcher',
	    'karma-firefox-launcher',
	    'karma-junit-reporter',
	    'karma-coverage'
	];

4.放入相应的代码到src以及test目录里。执行'karma start'命令, 浏览器将会被打开，然后执行相应的Test。效果如下图：   
![Karma in Chrome](/images/blog/karma-chrome.png)

完整样例代码：   
<https://github.com/blueshen/Karma-mocha-example>

##IntelliJ IDEA集成##
为了在项目中开发方便，那么在开发中集成到IDE中，会节省N多时间的。下面就先来说说于IDEA的集成。   
1.安装NodeJS插件： Settings --> IDE Settings --> Plugins --> Browse repositories --> NodeJS  选中，然后右键Download and Install进行安装。重启后成功安装。   
2.配置Karma Server: 从菜单Run --> Edit Configurations... -->点击 ‘+’新建一个Node.js类型的配置。出现以下的界面：    
![karma-node-server](/images/blog/karma-node-server.png)

其中：    
Name： 任意，本处为Karma node Server   
Path to Node: node可执行全路径。$NODE_PATH/bin/node     
Working Directory: 当前项目的跟路径   
Path to Node App JS File: 此处为karma的可执行全路径。   
Application Parameters: 要执行的命令，此处为start     
Environment variables: 就是环境变量了。此处我定义了CHROME_BIN来指出CHROME浏览器路径。  

3.配置Karma run     
同Karma Server，只是修改Application Parameters为run
![karma-node-run](/images/blog/karma-node-run.png)


配置成功后，运行Karma node Server可以看到浏览器就可以正常启动了。console也正确的输出。如同在terminal里执行一般。需要注意的是，本地开发时，需要将`singleRun=false`,也就是说执行完测试之后不退出。只有在CI环境下才用true。   
  
在浏览器启动之后，如果修改了源代码，Test能否自动执行呢？可以将`autoWatch=true`,这样当你修改代码后，保存后能自动执行，方便开发了。如果‘autoWatch=false’了，那么这时间就要执行Karma run了，用于在Karma Server上重新执行。   
   


