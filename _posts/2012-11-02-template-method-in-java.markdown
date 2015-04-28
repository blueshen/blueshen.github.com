---
layout: post
title: "设计模式：模板方法（template method） in java"
date: 2012-11-07 19:30
comments: true
categories: 设计模式
tags: [ Java, template method ]
---
**定义：**定义一个操作中的算法的框架，而将一些步骤延迟到子类中。使得子类可以不改变一个算法的结构即可重定义这个算法的某些特定步骤。   
简单的说，就是你首先定义一下算法的各部分之间的执行顺序或者调用关系，然后在子类中实现具体每一部分是如何实现的。   

采用什么算法作为例子呢？就是用**把动物放进冰箱**这个算法吧。  

<!--more-->
先定义一个抽象类:  
	
	public abstract class AbstractAlgorithm {

		public abstract void openFridgeDoor();

		public abstract void putAnimalInFridge();

		public abstract void closeFridgeDoor();
	
		public void execute(){
			this.openFridgeDoor();
			this.putAnimalInFridge();
			this.closeFridgeDoor();
		}
	}

这个类，定义了3个操作，打开冰箱门、把动物放进冰箱、关闭冰箱门，但并未实现，而留给子类来具体实现。`execute()`用来执行这一算法，它指定了各个操作之间的先后顺序。  

比如：我想把大象放进冰箱里：  

	public class PutElephantInFridge extends AbstractAlgorithm {

		@Override
		public void openFridgeDoor() {
			System.out.println("open the fridge door lightly");
		}

		@Override
		public void putAnimalInFridge() {
			System.out
				.println("try my best to put elephant in fridge,after 2 hours, I got it.");
		}

		@Override
		public void closeFridgeDoor() {
			System.out.println("close the fridge door...");
		}

	}
具体使用：

	PutElephantInFridge algo = new PutElephantInFridge();
	algo.execute();
好吧，执行一下算法就完成了。
下面吗？我想把猴子放进冰箱，好吧，新建一个类继承AbstractAlgorithm，然后重写相关步骤就可以了。

##模板方法 In JDK##
	
	java.io.InputStream, java.io.OutputStream, java.io.Reader，java.io.Writer      

	java.util.AbstractList, java.util.AbstractSet and java.util.AbstractMap
所有非抽象方法。

	javax.servlet.http.HttpServlet#doXXX()   
都默认返回一个`SC_METHOD_NOT_ALLOWED`类似的错误码，或者代码，要想使用，只有继承并且重写这些方法。


