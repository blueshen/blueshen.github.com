---
layout: post
title: "Qunit 简介"
date: 2013-03-22 15:21
comments: true
categories: nodejs
tags: [ qunit, jquery ]
---
#QUnit
QUnit是一个强大的JavaScript单元测试框架，用于调试代码。该框架是由jQuery团队的成员所开发，并且是jQuery的官方测试套件。任意正规JavaScript代码QUnit都能测试。   
[项目官网](http://qunitjs.com/)   
[文件下载地址](https://github.com/jquery/qunit)   
#建立测试程序
建立html测试页面，引入 `qunit.js` 和 `qunit.css` 这两个必需的文件。其中`qunit.js`是测试套件程序，`qunit.css`用于控制测试套件的结果显示的样式。    
	
	<!--sample.html:-->
	<html>
	<head>
	<meta charset="utf-8">
	  <title>QUnit basic example</title>
	  <link rel="stylesheet" href="./resources/qunit.css">
	<script type="text/javascript" src="./resources/jquery.js"></script>
	</head>
	<body>
	  <div id="qunit"></div>
	  <div id="qunit-fixture"></div>
	  <script src="./resources/qunit.js"></script>
	  <script>
	    test( "a basic test example", function() {
	      var value = "hello";
	      equal( value, "hello", "We expect value to be hello" );
	    });
	  </script>
	</body>
	</html>
<!--more-->
其中放置的文件及文件结构如下：     
  
	|-qunit-test   
	|  |-sample.html   
	|  |-resources   
	|  |    |-qunit.js   
	|  |    |-qunit.css
    |  |    |-jquery.js   

测试的结果会由`qunit.js`控制输出到页面代码中的`<div id="qunit"></div>`中。另外一个必不可少的元素是`<div id=""qunit-fixture""></div>`。在每个test执行完毕后，如果改动了该元素，会自动重置。`jquery.js`的引入是为了测试使用jQuery语法写的程序。   
在浏览器中打开sample1.html可以看到结果显示如下图所示：   
![qunit的用例运行显示结果](/images/blog/qunit-pic.png)   
#测试框架使用说明   
标题下面有一条横线，绿色表示全部用例正确，红色表示至少有一个用例错误。   
下面是3个checkbox。"Hide passed tests"点击后可以过滤掉通过的用例，只显示失败的用例。"Check for Globals"，用来检查window对象在test运行前后的变化，如果出现变化，则会报错。"No try-catch"用来显示测试用例中抛出的异常，当选中时直接将其死掉，不选中时则显示报错信息。对每个测试用例，标题中包含（x,y,z）表示总共有z个断言，y个是正确的，x个是错误的。   
#断言   
- **ok( truthy [, message ] )**   判断是否为true   
- **equal( actual, expected [, message ] )**    判断actual==expected   
- **deepEqual( actual, expected [, message ] )**    判断actual===expected
   
用例如下：   
	test("assertion",function(){   

		ok( true, "true succeeds" );
	    ok( NaN, "NaN fails" );
	   
	    equal( 0, 0, "0, 0 : equal succeeds" );
		equal( "", 0, "Empty, 0: equal succeeds" );
	    equal( null, "", "null, empty: equal fails" );
	  
	    var obj = { foo: "bar" };
	    deepEqual( obj, { foo: "bar" }, "Two objects can be the same in value" );
		equal( "", 0, "Empty, 0: equal succeeds" );
	});    

#测试同步代码   
在同步代码的测试中，有两种方式：   
-   test( name, expected, fucntion(){...})：expceted指assertion的数量。   
-   test( name, function(){expected(amount);...})：在function中增加expected(amount)，amount表示assertiong的数量。   

test()是常规的测试用例，并且默认是同步的，这意味着他们是一个接一个的运行。expected()最有价值的地方在于callback函数的测试。当callback函数因为任何原因不能执行时，会造成实际断言的数量不等于expected值，这时会有额外的错误提示。   
	
	test( "a test", 2, function() {
		ok( true, "sucess" );
		ok( false, "fail" );
	});   

	test( "a test", function() {
	  expect( 2 );
	 
	  function calc( x, operation ) {
	    return operation( x );
	  }
	 
	  var result = calc( 2, function( x ) {
	    ok( true, "calc() calls operation function" );
	    return x * x;
	  });
	 
	  equal( result, 4, "2 square equals 4" );
	});


#测试异步代码
对Ajax请求或通过setTimeout()或sestInterval()调用的方法，需要使用异步测试函数asyncTest()。   

	asyncTest( "asynchronous test: one second later!", function() {
	  expect( 1 );
	 
	  setTimeout(function() {
	     ok( true, "Passed and ready to resume!" );
	     start();
	  }, 1000);
	});      

#用户行为的测试
测试用户行为时，无法使用一个函数就搞定，通常需要使用一个匿名函数绑定到元素的事件上来模拟。事件的触发使用`trigger()`或者`triggerHandler()`来实现。	
   
	test( "div click test", 1, function() {
	  var $body = $( "#qunit-fixture" );
	 
	  $body.bind( "click", function() {
		ok( true, "body was clicked!" );
	  });
	 
	  $body.trigger( "click" );
	});
下面是Qunit中的一个demo例子，其中模拟了一个key的记录器`KeyLogger()`，在test中初始化了一个事件event，并且使触发了两次，：   

	function KeyLogger( target ) {
	  if ( !(this instanceof KeyLogger) ) {
		return new KeyLogger( target );
	  }
	  this.target = target;
	  this.log = [];
	 
	  var self = this;
	 
	  this.target.bind( "keydown", function( event ) {
		self.log.push( event.keyCode );
	  });
	}
	test( "keylogger api behavior", function() {
	 
	  var event,
		  $doc = $( document ),
		  keys = KeyLogger( $doc );
	 
	  // trigger event
	  event = $.Event( "keydown" );
	  
	  event.keyCode = 'A';
	  $doc.trigger( event );
	  $doc.trigger( event );
	 
	  // verify expected behavior
	  equal( keys.log.length, 2, "2 key was logged" );
	  equal( keys.log[ 0 ], 'A', "correct key was logged" );
	 
	});   
#模块化   
为了使自己的用例的顺序更加富有逻辑性，可以使用module()函数对用例进行分组。对出现在某个module（）后面的所有用例都被分在该组中。   
   
	module( "group a" );
	test( "a basic test example", function() {
	  ok( true, "this test is fine" );
	});
	test( "a basic test example 2", function() {
	  ok( true, "this test is fine" );
	});
	 
	module( "group b" );
	test( "a basic test example 3", function() {
	  ok( true, "this test is fine" );
	});
	test( "a basic test example 4", function() {
	  ok( true, "this test is fine" );
	});

除了可以进行分组之外，module()还可以从测试用例中抽取通用的代码，用可选的第二个参数来定义每个test在运行之前、之后的函数。   
   
	module( "module", {
	  setup: function() {
	    ok( true, "one extra assert per test" );
	  }, teardown: function() {
	    ok( true, "and one extra assert after each test" );
	  }
	});
	test( "test with setup and teardown", function() {
	  expect( 3 );
	  ok( true, "test" );
	});

#推荐使用的框架程序	
最上面建立的测试框架为用于学习时建立的demo框架。真正在使用中，为了方便我们习惯通过外部引入的方式来进行测试用例的书写。如下面所示，直接在项目中引入项目代码`myProject.js`和测试代码`myTests.js`。 

	<!--sample-framework.html:-->
	<html>
	<head>
	<meta charset="utf-8">
	  <title>QUnit basic example</title>
	  <link rel="stylesheet" href="./resources/qunit.css">
	</head>
	<body>
	  <div id="qunit"></div>
	  <div id="qunit-fixture"></div>
	  <script src="./resources/qunit.js"></script>
	  <script type="text/javascript" src="./resources/jquery.js"></script>
	    <!-- 项目代码 -->
	  <script type="text/javascript" src="myProject.js"></script>
		<!-- 测试代码 -->
	  <script type="text/javascript" src="myTests.js"></script>

	</body>
	</html>

---
参考文献：<http://qunitjs.com/cookbook/>   
感谢：lizejun

