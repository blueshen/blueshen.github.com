---
layout: post
title: "设计模式：原型（prototype） in java"
date: 2012-11-07 20:41
comments: true
categories: 设计模式
tags: [ Java, prototype, Clone ]
---
**定义：**用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。  
基本上，可以就是一个clone方法，通过这个方法进行对象的拷贝。 

Java中的原型模式：

	public class ProtoTypeClass implements Cloneable {

		@Override
		public ProtoTypeClass clone(){
			ProtoTypeClass cloneObject = null;
			try{
				cloneObject = (ProtoTypeClass) super.clone();
			}catch (Exception e) {
				// TODO: handle exception
			}
			return cloneObject;
		}
	}
上面就是实现了原型模式。不过Java在提供了Cloneable这一接口方便实现原型模式的同时，也带来了一些不容易注意到的问题。

* clone时，构造函数不会执行
* 浅拷贝与深拷贝

这两个问题是需要时刻注意的。由于本文主要不是讲Cloneable,所以另辟专题吧。