---
layout: post
title: "Mocha简介"
date: 2013-03-18 20:44
comments: true
categories: nodejs
---
##Mocha##
Mocha作为一种咖啡名，应该是广为人知的，中文翻译为**摩卡**。在这里，我们介绍的是一个JavaScript Test Framework，它用于对NodeJS、JavaScript进行单元测试。  
Mocha是一个功能丰富的Javascript测试框架，能够运行在node和浏览器上，并且有丰富的报表支持。   
项目主页：<http://visionmedia.github.com/mocha/>    
##安装##
	npm install -g mocha

##一个简单的样例##

	$ mkdir test
	$ cd ..
	$ mocha test/test.js

	var assert = require("assert")
	describe('Array', function(){
  	describe('#indexOf()', function(){
    	it('should return -1 when the value is not present', function(){
      	assert.equal(-1, [1,2,3].indexOf(5));
      	assert.equal(-1, [1,2,3].indexOf(0));
    	})
  	})
	})

	$  mocha

 	 .

 	 ✔ 1 test complete (1ms)
<!--more-->
##Assertions##
在Java Unit Test中类似JUNIT，TestNG提供了不少的Assert函数。同样的，mocha也有很多选择。而这些并不属于mocha的一部分。

* [should.js](http://github.com/visionmedia/should.js)
* [expect.js](https://github.com/LearnBoost/expect.js)
* [chai](http://chaijs.com/)
* [better-assert](https://github.com/visionmedia/better-assert)

##测试同步代码##

	describe('Array', function(){
  	describe('#indexOf()', function(){
    	it('should return -1 when the value is not present', function(){
      	[1,2,3].indexOf(5).should.equal(-1);
      	[1,2,3].indexOf(0).should.equal(-1);
    	})
  	})
	})
##测试异步代码##
添加一个回调函数，通常称为done,给it。mocha就会知道应该等待操作完成。

	describe('User', function(){
  	describe('#save()', function(){
    	it('should save without error', function(done){
      	var user = new User('Luna');
      	user.save(done);
    	})
  	})
	})

##类似与JUNIT的函数

* before : 在所有测试执行之前
* after ： 在所有测试执行之后
* beforeEach ： 每个测试之前
* afterEach ：每个测试之后

##mocha指令##

	Usage: mocha [debug] [options] [files]

	Commands:

  	init <path>
  	initialize a client-side mocha setup at <path>

	Options:

  	-h, --help                      帮助信息
  	-V, --version                   版本信息
  	-r, --require <name>            依赖的module
  	-R, --reporter <name>           使用的报告模式
  	-u, --ui <name>                 用什么接口(bdd|tdd|exports)
  	-g, --grep <pattern>            执行匹配 <pattern>的测试
  	-i, --invert                    --grep 相反的测试
  	-t, --timeout <ms>              超时毫秒数 [2000]
  	-s, --slow <ms>                 "slow" 测试的门槛 [75]
  	-w, --watch                     查看文件的变化，如true,则变化后自动运行。
  	-c, --colors                    启用colors
  	-C, --no-colors                 禁用colors
  	-G, --growl                     启用growl notification
  	-d, --debug                     启用debug
  	-b, --bail                      只对第一个报错的TEST感兴趣
  	--recursive                     递归执行
  	--debug-brk                     enable node's debugger breaking on the first line
  	--globals <names>               allow the given comma-delimited global [names]
  	--ignore-leaks                  ignore global variable leaks
  	--interfaces                    显示可用的接口
  	--reporters                     显示可用的报表列表
  	--compilers <ext>:<module>,...  使用指定的module来编译文件


##报表##

	dot - dot matrix
    	doc - html documentation
    	spec - hierarchical spec list
    	json - single json object
    	progress - progress bar
    	list - spec-style listing
    	tap - test-anything-protocol
    	landing - unicode landing strip
    	xunit - xunit reportert
    	teamcity - teamcity ci support
    	html-cov - HTML test coverage
    	json-cov - JSON test coverage
    	min - minimal reporter (great with --watch)
    	json-stream - newline delimited json events
    	markdown - markdown documentation (github flavour)
    	nyan - nyan cat!
