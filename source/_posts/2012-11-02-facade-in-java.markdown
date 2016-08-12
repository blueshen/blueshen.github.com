---
layout: post
title: "设计模式：门面（Facade） in java"
date: 2012-11-07 20:00
comments: true
categories: 设计模式
tags: [ Java, Facade, slf4j ]
---
**定义：**为子系统中的一组接口提供一个一致的界面，Facade模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。   
就是说，Facade提供了一个统一的接口，掩盖下层系统的复杂性，用户用起来更加的方便。

以医院的例子，做个比喻：  
<!--more-->  
 
在无接待员的时候，病人要做业务，好复杂啊，好累！

![无接待员](/images/blog/facade-hospital1.png)

有了接待员，各种就医流程好流畅的说。因为有接待员与各个部门打交道。

![有接待员](/images/blog/facade-hospital2.png)  

这个例子很好的说出了facade的作用。甚至都不需要代码来表达了。

门面模式的优点：

　　●松散耦合

　　门面模式松散了客户端与子系统的耦合关系，让子系统内部的模块能更容易扩展和维护。

　　●简单易用

　　门面模式让子系统更加易用，客户端不再需要了解子系统内部的实现，也不需要跟众多子系统内部的模块进行交互，只需要跟门面类交互就可以了。

　　●更好的划分访问层次

　　通过合理使用Facade，可以帮助我们更好地划分访问的层次。有些方法是对系统外的，有些方法是系统内部使用的。把需要暴露给外部的功能集中到门面中，这样既方便客户端使用，也很好地隐藏了内部的细节    

##门面模式 in JDK##
这个具体的例子，我首先想到的就是[slf4j](http://www.slf4j.org/)这个日志框架。通过名字Simple Logging Facade for Java (SLF4J)就知道是采用的Facade模式了。下面是其官方的介绍：   
The Simple Logging Facade for Java or (SLF4J) serves as a simple facade or abstraction for various logging frameworks, e.g. java.util.logging, log4j and logback, allowing the end user to plug in the desired logging framework at deployment time.   
也就是说，他屏蔽了各种日志框架的差异，提供了一个统一的日志接口给用户使用。不得不说，[slf4j](http://www.slf4j.org/)很好用，推荐！

