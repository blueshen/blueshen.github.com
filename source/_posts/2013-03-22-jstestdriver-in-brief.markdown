---
layout: post
title: "JsTestDriver 简介"
date: 2013-03-22 16:11
comments: true
categories: nodejs
tags: [ jsTestDriver, nodejs, test-runner]
---
#jsTestDriver
jsTestDriver是一个JavaScript单元测试工具，易于与持续构建系统相集成并能够在多个浏览器上执行运行测试，轻松实现TDD（测试驱动开发）风格的开发。当在项目中配置好js-test-driver以后，如同junit测试java文件一般，js-test-driver可以直接通过直接运行js文件，来对js文件单元测试。   
![alt jsTestDriver框架](/images/blog/jsTestDriver-framework.jpg)
<!--more-->
##### 在Intellij IDEA中安装JsTestDriver####
* 打开IDEA编辑器，点击**File**，点击下拉列表中的**setting**，进入IDEA设置对话框   
* 在搜索框中键入**plugins**，在搜索结果中选择**plugins**这一项  
* 点击**Browse Repositories**，在弹出的列表中搜索jsTestDriver。   
* 右击jsTestDriver插件，选择**Download and Install**.   
 
![alt jsTestDriver插件安装](/images/blog/idea-install-jstestdriver-plugin-dialog.png)   
##### 在IDEA中使用jsTestDriver运行js测试代码#####
* 在IDEA中新建一个空的工程，在工程目录下新建代码包test
* 在src代码包中新建Greeter.js代码如下：  

    myapp = {};  
    myapp.Greeter = function() { };  
    myapp.Greeter.prototype.greet = function(name) {
    return "Hello " + name + "!";  
    };   
* 在test代码包中新建GreeterTest.js,代码如下

    GGdTestCase("GreeterTest", {
    "test greet": function() {
        var greeter = new myapp.Greeter();
        assertEquals("Hello World!", greeter.greet("World"));
    },
    "test greet null": function() {
        var greeter = new myapp.Greeter();
        //assertNull(greeter.greet(null));
        assertTrue(true);
    }
    });  
* 在项目主文件夹中新建配置文件greeter.jstd,文件内容如下：

    load:  
  -- src/Greeter.js  
  --test/GreeterTest.js  
* 启动jsTestDriver Server  
  ![alt jsTestDriver server](/images/blog/jsTestDriver-server.jpg)
* 打开本地浏览器，访问url http://localhost:9876/capture
* 运行greeter.jstd
    
##### 在Eclipse中安装jsTestDriver#####
* 在**Help**中的**Install new software**中，添加一个update site ：http://js-test-driver.googlecode.com/svn/update/  
* 安装完毕后，重启Eclipse，新建一个空的java项目
* 在java项目中添加test代码包，在src中新建src.js,其代码如IDEA中的Greeter.js一样。
* 在test中添加test.js，其代码和IDEA中GreeterTest.js一样。  
* 在项目目录中添加配置文件jsTestDriver.conf，其内容为

    load:  
    -- src/*.js  
    -- test/*.js  
目录结构如图：  
![alt 目录结构](/images/blog/eclipse-jstestDriver.jpg)
* 配置Run Configuration，新建一个Js Test Driver Test, 选择好项目和相应的配置文件。
* 启动jsTestDriver服务器，然后用浏览器去访问http://127.0.0.1:4244/capture，这样就可以在
浏览器中执行我们的js测试脚本了。  
![alt eclipse执行结果](/images/blog//eclipse-test.jpg)  
我们可以再eclipse中配置jsTestDriver的相关项，如图：  
![alt eclipse配置jsTestDriver](/images/blog//eclipse-js-setting.jpg)

---
参考文献：<http://code.google.com/p/js-test-driver/>   
感谢：youthflies