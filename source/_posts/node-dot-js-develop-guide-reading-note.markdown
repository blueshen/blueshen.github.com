---
layout: post
title: "《Node.js开发指南》读书笔记"
date: 2013-03-11 16:53
comments: true
categories: javascript
tags: [ NodeJS, express, 读书笔记 ]
---
###书籍信息##
Amazon: [NodeJS开发指南](http://www.amazon.cn/Node-js%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97-%E9%83%AD%E5%AE%B6%E5%AE%9D/dp/B008HN793I)

PDF: [免费下载](http://azrael.ihorsley.com/wordpress/wp-content/uploads/2012/11/Node.js%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97_%E4%B8%AD%E6%96%87%E6%AD%A3%E7%89%88.pdf)

###NodeJS简介###
是NodeJS的出现，让JavaScript在服务器端得以使用，重新焕发了生机。而不仅仅像大家所认为的，只是一个客户端脚本语言。  
由于JavaScript自身的脚本语言特性，造成开发混乱，难以维护。CommonJS对这个进行了规范。像NodeJS,ringojs都是对这一规范的具体实现。
CommonJS规范包括：  

* 模块（modules）
* 包（packages）
* 系统（system）
* 二进制（binary）
* 控制台（console）
* 编码（encodings）
* 文件系统（filesystems）
* 套接字（sockets）
* 单元测试（unit testing）
<!--more-->
###NodeJS的模块与包###
模块（module）和包(package)是NodeJS的基本。并且都是参照CommonJS标准来实现的。如果项目有一定的规模，势必要把各种功能模块进行切分，然后再组装起来。这也正式所有服务器端的通用做法。然而，在NodeJS中怎么实现模块之间的调用呢，这里是使用require函数的。模块和包通常区分不是很明确，可以认为是一致的。
####1.什么是模块？

	var http = require("http");
其中http就是nodeJs中的一个核心模块. 像Java中的import一样，这里是使用require来引入这个模块。

####2.创建与发布模块
NodeJS提供了exports和require两个对象来完成，exports用于公开模块的接口，require用于获取外部模块的接口。
如创建一个module.js:   

	var name;
	exports.setName=function(thyName){
	     name=thyName;
         }
         exports.sayHello = function(){
	    console.log('Hello '+name);
	}
在同一个目录下，再创建一个getmodule.js: 

	var mymodule = require('./module');
	myModule.setName('shenyanchao');
	myModule.sayHello();
运行后的结果：

	Hello shenyanchao
这就是一个简单的模块发布与调用关系。   
####3.包（package）
包是对模块的更进一步的抽象。类似与Java的类库概念。当包便多，甚至依赖很复杂的时候，就需要一个管理工具，就像是Java的Maven用来管理Jar包一样。NodeJs用NPM（Node Packages Manager）来发布、更新、依赖管理和版本控制。  
直观上看，NodeJS的包是一个目录，并且包含一个package.json文件。一个符合CommonJS的包应有以下的特征： 

* package.json在包的顶层目录下；
* 二进制可执行文件在bin目录下；
* JS代码在lib目录下；
* 文档在doc目录下；
* 单元测试在test下；
这就相当于对包的目录结构进行了一个定义，类似于J2EE的规范一样，减少大家的学习成本，什么东西放在哪儿都一清二楚。如果在github或者googlecode上看开源项目，绝对都是这样的结构。  

模块与文件是一一对应的。文件不仅可以是 JavaScript 代码或二进制代码,还可以是一个文件夹。最简单的包,就是一个作为文件夹的模块。建立一个叫做 somepackage 的文件夹,在其中创建 index.js,内容如下:

	exports.hello = function() {
	   console.log('Hello.');
	};
然后在 somepackage 之外建立 getpackage.js,内容如下:   

	var somePackage = require('./somepackage');
	somePackage.hello();

运行 node getpackage.js,控制台将输出结果 Hello。   
我们使用这种方法可以把文件夹封装为一个模块,即所谓的包。包通常是一些模块的集合,在模块的基础上提供了更高层的抽象,相当于提供了一些固定接口的函数库。通过定制package.json,我们可以创建更复杂、更完善、更符合规范的包用于发布。   
**package.json**
在somepackage 文件夹下,我们创建一个叫做 package.json 的文件,内容如下所示:  

	{
	    "main" : "./lib/interface.js"
	}
然后将 index.js 重命名为 interface.js 并放入 lib 子文件夹下。以同样的方式再次调用这个包,依然可以正常使用。   
NodeJS在调用某个包时,会首先检查包中 package.json 文件的 main 字段,将其作为包的接口模块,如package.json 或 main 字段不存在,会尝试寻找 index.js 或 index.node 作为包的接口。  
package.json 是 CommonJS 规定的用来描述包的文件,完全符合规范的 package.json 文件应该含有以下字段。   
name:包的名称,必须是唯一的,由小写英文字母、数字和下划线组成,不能包含空格。   
description:包的简要说明。   
version:符合语义化版本识别 规范的版本字符串。  
keywords:关键字数组,通常用于搜索。   
maintainers:维护者数组,每个元素要包含 name、email (可选) web (可选)字段。  
contributors:贡献者数组,格式与maintainers相同。包的作者应该是贡献者数组的第一个元素。   
bugs:提交bug的地址,可以是网址或者电子邮件地址。   
licenses:许可证数组,每个元素要包含 type (许可证的名称)和 url (链接到许可证文本的地址)字段。   
repositories:仓库托管地址数组,每个元素要包含 type(仓库的类型, git )如url (仓库的地址)和 path (相对于仓库的路径,可选)字段。   
下面是mocha的package.json:  
	
	{
 	 "name": "mocha",
 	 "version": "1.8.1",
  	"description": "simple, flexible, fun test framework",
 	 "keywords": [
   	 "mocha",
    	"test",
   	 "bdd",
   	 "tdd",
   	 "tap"
  	],
 	 "author": {
   	 "name": "TJ Holowaychuk",
    	"email": "tj@vision-media.ca"
 	 },
  	"repository": {
    	"type": "git",
    	"url": "git://github.com/visionmedia/mocha.git"
 	 },
  	"main": "./index",
  	"bin": {
    	"mocha": "./bin/mocha",
    	"_mocha": "./bin/_mocha"
  	},
  	"engines": {
    	"node": ">= 0.4.x"
  	},
  	"scripts": {
    	"test": "make test-all"
 	 },
  	"dependencies": {
    	"commander": "0.6.1",
    	"growl": "1.7.x",
    	"jade": "0.26.3",
    	"diff": "1.0.2",
    	"debug": "*",
    	"mkdirp": "0.3.3",
    	"ms": "0.3.0"
  	},
  	"devDependencies": {
   	 "should": "*",
    	"coffee-script": "1.2"
 	 },
  	"readme": "..."
	}
也就是说，这里面提供了完善的信息来告诉npm，怎么样安装、升级、传播。   
如执行：  

	npm install -g mocha
那么，npm将会依据json提供的信息来进行管理。
####4.npm的本地模式与全局模式
npm默认会从http://npmjs.org上搜索并下载包，并将包安装在当前目录的node_modules子目录下。这种就称为本地模式。也就意味着只能在当前目录使用。如果想在全部地方可用，那就用`-g`参数。这样包就会安装到NODE_PATH里了，在任何目录都可以使用了。g应该就是global的缩写，很容易记。  

###模块（modules）的加载机制
前面，已经知道模块加载是通过require来进行的。NodeJS的模块可以分为2大类，一类是核心模块、一类是文件模块。核心模块有最高的优先级，如有模块命名冲突，NodeJS总是优先加载核心模块。  
那么，文件模块是如何加载的呢？
####按路径加载模块####
1.如果require按“/”开头，那就是绝对路径进行加载。如require('/home/shenyanchao/module'),将会按照以下优先级尝试加载 /home/shenyanchao/module.js、/home/shenyanchao/module.json、/home/shenyanchao/module.node。   
2.如果require按“./”或者“../”开头，则是依相对路径来查找模块，这种较为常见。  
3.对于核心模块,比如require('http')，nodeJS是怎么找到的呢，自然是通过NODE_PATH目录加载的。那么对于文件模块，如果不用绝对路径已经相对路径，那么该如何查找呢？   
如果，使用require('mymodule'),那么NodeJS将首先在当前目录的node_modules目录内进行尝试加载。如果没有找到，那么将会到当前目录的上一级目录的node_modules继续查找，并反复执行，直到根目录为止。   
例如： 我们在/home/shenyanchao/develop/app.js中使用require('mymodule')，NodeJS的查找路径如下：  

* /home/shenyanchao/develop/node_modules/mymodule.js
* /home/shenyanchao/node_modules/mymodule.js
* /home/node_modules/mymodule.js
* /node_modules/mymodule.js

这个时候，明白了加载机制，就可以返回来，看一下模块的本地模式于全局模式来。当以本地模式`npm install mocha`的时候，会在当前目录建立一个node_modules目录，这就保证了系统内使用require('mocha')时，能够直接使用。而`npm install -g mocha`相当于把mocha安装到NODE_PATH，这样就使用类似于加载核心模块的形式进行加载了。

###express: JS的MVC框架###
这里介绍来一个强大的Web application Framework for Node。用于进行WEB项目的开发。类似于Java的SpringFramework。很轻量级，简单易用。   
express将NodeJS的开发，推向了一个新的高度。很有兴趣，待研究！    
express主页：<http://expressjs.com/>

	