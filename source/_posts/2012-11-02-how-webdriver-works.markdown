---
layout: post
title: "Selenium WebDriver 工作原理"
date: 2012-11-02 18:26
comments: true
categories: selenium
tags: [ webdriver , selenium  ]
---
WebDriver与之前Selenium的JS注入实现不同，直接利用了浏览器native support来操作浏览器。所以对于不同平台，不同的浏览器，必须依赖一个特定的浏览器的native component来实现把WebDriver API的调用转化为浏览器的native invoke。   
<!--more-->

在我们new一个WebDriver的过程中，Selenium首先会确认浏览器的native component是否存在可用而且版本匹配。接着就在目标浏览器里启动一整套Web Service，这套Web Service使用了Selenium自己设计定义的协议，名字叫做The WebDriver Wire Protocol。这套协议非常之强大，几乎可以操作浏览器做任何事情，包括打开、关闭、最大化、最小化、元素定位、元素点击、上传文件等等等等。

WebDriver Wire协议是通用的，也就是说不管是FirefoxDriver还是ChromeDriver，启动之后都会在某一个端口启动基于这套协议的Web Service。例如FirefoxDriver初始化成功之后，默认会从http://localhost:7055开始，而ChromeDriver则大概是http://localhost:46350之类的。接下来，我们调用WebDriver的任何API，都需要借助一个ComandExecutor发送一个命令，实际上是一个HTTP request给监听端口上的Web Service。在我们的HTTP request的body中，会以WebDriver Wire协议规定的JSON格式的字符串来告诉Selenium我们希望浏览器接下来做社么事情。

WebDriver的工作原理图：

![WebDriver工作原理图](/images/blog/webdriver-works.png)

从上图中我们可以看出，不同浏览器的WebDriver子类，都需要依赖特定的浏览器原生组件，例如Firefox就需要一个add-on名字叫webdriver.xpi。而IE的话就需要用到一个dll文件来转化Web Service的命令为浏览器native的调用。另外，图中还标明了WebDriver Wire协议是一套基于RESTful的web service。

关于WebDriver Wire协议的细节，比如希望了解这套Web Service能够做哪些事情，可以阅读Selenium官方的协议文档， 在Selenium的源码中，我们可以找到一个HttpCommandExecutor这个类，里面维护了一个Map<String, CommandInfo>，它负责将一个个代表命令的简单字符串key，转化为相应的URL，因为REST的理念是将所有的操作视作一个个状态，每一个状态对应一个URI。所以当我们以特定的URL发送HTTP request给这个RESTful web service之后，它就能解析出需要执行的操作。截取一段源码如下：   

	nameToUrl = ImmutableMap.<String, CommandInfo>builder()  
        .put(NEW_SESSION, post("/session"))  
        .put(QUIT, delete("/session/:sessionId"))  
        .put(GET_CURRENT_WINDOW_HANDLE, get("/session/:sessionId/window_handle"))  
        .put(GET_WINDOW_HANDLES, get("/session/:sessionId/window_handles"))  
        .put(GET, post("/session/:sessionId/url"))  
  			// The Alert API is still experimental and should not be used.  
        .put(GET_ALERT, get("/session/:sessionId/alert"))  
        .put(DISMISS_ALERT, post("/session/:sessionId/dismiss_alert"))  
        .put(ACCEPT_ALERT, post("/session/:sessionId/accept_alert"))  
        .put(GET_ALERT_TEXT, get("/session/:sessionId/alert_text"))  
        .put(SET_ALERT_VALUE, post("/session/:sessionId/alert_text"))  

可以看到实际发送的URL都是相对路径，后缀多以/session/:sessionId开头，这也意味着WebDriver每次启动浏览器都会分配一个独立的sessionId，多线程并行的时候彼此之间不会有冲突和干扰。例如我们最常用的一个WebDriver的API，getWebElement在这里就会转化为/session/:sessionId/element这个URL，然后在发出的HTTP request body内再附上具体的参数比如by ID还是CSS还是Xpath，各自的值又是什么。收到并执行了这个操作之后，也会回复一个HTTP response。内容也是JSON，会返回找到的WebElement的各种细节，比如text、CSS selector、tag name、class name等等。以下是解析我们说的HTTP response的代码片段：

	try {  
        response = new JsonToBeanConverter().convert(Response.class, responseAsText);  
      } catch (ClassCastException e) {  
        if (responseAsText != null && "".equals(responseAsText)) {  
          // The remote server has died, but has already set some headers.  
          // Normally this occurs when the final window of the firefox driver  
          // is closed on OS X. Return null, as the return value _should_ be  
          // being ignored. This is not an elegant solution.  
          return null;  
        }  
        throw new WebDriverException("Cannot convert text to response: " + responseAsText, e);  
      } //...  

相信总结道这里，应该对WebDriver的运行原理应该清楚了！其实挺佩服这一套RESTful web service的设计。

原文：<http://blog.csdn.net/ant_yan/article/details/7970793>
