---
layout: post
title: "Jasmine 简介"
date: 2013-03-22 15:47
comments: true
categories: nodejs
tags: [ nodejs, jasmine]
---
##jasmine测试框架简介##
**jasmine**是一种javascript测试框架，它既可以在html文件中运行，也可以和jsTestDriver整合，在jsTestDriver中运行。
###jasmine的简单语法###
一个基本的jasmine测试用例如下：  
    describe("A suite", function() {  
  		it("contains spec with an expectation", function() {  
    	expect(true).toBe(true);  
  		});  
	});   
####describe方法####
describe方法标志着一个测试集(test suite)的开始，这个方法有两个参数，一个字符串String，一个方法function；字符串用来描述我们这个test suite，function里的东西就是测试代码，它们就是suite。  
<!--more-->
####it方法####
jasmine中用方法it来开始specs（我理解成测试点，一个测试suite里可以有很多spec）。it方法和describe方法类似，同样有两个参数，一个String，一个function；String用来描述测试点（spec），function是具体的测试代码。一个测试点(spec)可以包含多个expections（其实是断言的意思）。 
####expectations####
断言可以返回为true或者false。全部的断言返回true这个测试点就通过，一个或者多个断言返回false这个测试点就不通过。  
describe和it都是方法，我们可以自定义一些变量，在describe中定义的变量，在it方法中可以直接使用。  
    describe("A suite is just a function", function() {
	var a;

	it("and so is a spec", function() {
    	a = true;

    	expect(a).toBe(true);
	});
	});  
####Matchers####
一个一个的测试点们由expect开头，后面跟着一个我们需要测试的变量，如上面的a，然后跟着一个Matcher方法（我理解成校验规则），Matcher方法带着一个期望值，如上面的true。Matchers方法返回true或者false，它决定着测试点（spec）是否通过。所有的Matchers方法都能在mathcer前面加上not来进行否定断言，如`expect(a).not.toBe(true);  

jasmine中有很多Matchers方法供我们使用，当然我们也可以定义自己的Matchers方法。
	describe("Included matchers:", function() {

		it("The 'toBe' matcher compares with ===", function() {
    		var a = 12;
    		var b = a;

    		expect(a).toBe(b);
    		expect(a).not.toBe(null);
  		});  
		//上面的例子，比较a、b是否相等；验证a是否不是空。 

		it("should work for objects", function() {
      		var foo = {
        		a: 12,
        		b: 34
      		};
      		var bar = {
        		a: 12,
       	 		b: 34
      		};
      		expect(foo).toEqual(bar);
    	});
		//上面的例子比较了两个对象是否相等
	});

	it("The 'toMatch' matcher is for regular expressions", function() {
    	var message = 'foo bar baz';

    	expect(message).toMatch(/bar/);
    	expect(message).toMatch('bar');
    	expect(message).not.toMatch(/quux/);
	});
	//也可以使用正则表达式

	it("The 'toBeDefined' matcher compares against `undefined`", function() {
    	var a = {
      		foo: 'foo'
    	};

    	expect(a.foo).toBeDefined();
    	expect(a.bar).not.toBeDefined();
	});
	//验证变量是否被定义

	it("The 'toBeNull' matcher compares against null", function() {
    	var a = null;
   	 	var foo = 'foo';

    	expect(null).toBeNull();
    	expect(a).toBeNull();
    	expect(foo).not.toBeNull();
	});
	//验证是否为空

	it("The 'toBeTruthy' matcher is for boolean casting testing", function() {
    	var a, foo = 'foo';

    	expect(foo).toBeTruthy();
    	expect(a).not.toBeTruthy();
	});

	it("The 'toBeFalsy' matcher is for boolean casting testing", function() {
    	var a, foo = 'foo';

    	expect(a).toBeFalsy();
    	expect(foo).not.toBeFalsy();
	});
	//变量是否能够转化成boolean变量？ 不太确定

	it("The 'toContain' matcher is for finding an item in an Array", function() {
    	var a = ['foo', 'bar', 'baz'];

    	expect(a).toContain('bar');
    	expect(a).not.toContain('quux');
	});
	//是否包含
	it("The 'toBeLessThan' matcher is for mathematical comparisons", function() {
    	var pi = 3.1415926, e = 2.78;

    	expect(e).toBeLessThan(pi);
    	expect(pi).not.toBeLessThan(e);
	});

	it("The 'toBeGreaterThan' is for mathematical comparisons", function() {
    	var pi = 3.1415926, e = 2.78;

    	expect(pi).toBeGreaterThan(e);
    	expect(e).not.toBeGreaterThan(pi);
	});
	//数学大小的比较

	it("The 'toBeCloseTo' matcher is for precision math comparison", function() {
    var pi = 3.1415926, e = 2.78;

    expect(pi).not.toBeCloseTo(e, 2);
    expect(pi).toBeCloseTo(e, 0);
	});
	//两个数值是否接近，这里接近的意思是将pi和e保留一定小数位数后，是否相等。（一定小数位数：默认为2，也可以手动指定）

	it("The 'toThrow' matcher is for testing if a function throws an exception", function() {
    	var foo = function() {
      	return 1 + 2;
    	};
    	var bar = function() {
      		return a + 1;
    	};

    	expect(foo).not.toThrow();
    	expect(bar).toThrow();
		});
	}); 
	//测试一个方法是否抛出异常  

####Setup和Teardown方法####
为了代码简洁，减少重复性的工作，jasmine提供`beforeEach`和`afterEach`方法。`beforeEach`会在每个spec之前执行，`after`会在每个spec之后执行，类似于selenium中的`beforeMethod`和`afterMethod`方法。  
    describe("A spec (with setup and tear-down)", function() {
  		var foo;

  		beforeEach(function() {
    		foo = 1;
  		});

  		afterEach(function() {
    		foo = 0;
  		});

  		it("is just a function, so it can contain any code", function() {
    		expect(foo).toEqual(1);
  		});

  		it("can have more than one expectation", function() {
    		expect(foo).toEqual(1);
    		expect(true).toEqual(true);
  		});
	});  
另外describe和it作为方法是可以嵌套的，也就是describe中可以出现子describe和it。  
####禁用某些spec和suites####
在测试中，我们可能需要禁用一些suites和spec，方法是使用xdescribe和xit方法，这些测试的方法会被忽略，不计入统计结果。  
####The Runner and Reporter####
Jasmine是用javascript实现的，所以它也必须在javascript的环境中运行，最简单的环境也就是一个web页面。所有的spec都可以在这个页面中运行，这个页面就叫做Runner。  

Jasmine通过下面的js代码来展现spec运行结果：  
	var htmlReporter = new jasmine.HtmlReporter(); //创建一个HTMLReporter
	jasmineEnv.addReporter(htmlReporter);  

	jasmineEnv.specFilter = function(spec) {  //一个过滤器，允许我们点击单个的suites，单独运行
	return htmlReporter.specFilter(spec);
	};  


  	var currentWindowOnload = window.onload;   //页面加载完毕后，执行所有的test。
  	window.onload = function() {
    	if (currentWindowOnload) {
      		currentWindowOnload();
    	}

    	document.querySelector('.version').innerHTML = jasmineEnv.versionString();
    	execJasmine();
  	};

  	function execJasmine() {
    		jasmineEnv.execute();
  	}
	})();

---
参考文献：<http://pivotal.github.com/jasmine/>   
感谢：youthflies










