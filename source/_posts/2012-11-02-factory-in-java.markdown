---
layout: post
title: "设计模式:工厂（factory） in java"
date: 2012-11-07 17:43
comments: true
categories: 设计模式
tags: [ Java, Factory ]
---
工厂模式，直接按名字来说，就是负责专门生产产品的。   
大致分为3类： 

* 工厂方法
* 简单工厂
* 抽象工厂

同时，也有人认为简单工厂只是工厂方法的一种特列，那么就分为两种了。本文就按3种分别进行介绍了。   
###工厂方法###
**定义：**定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法让一个类的实例化延迟到其子类。  
<!--more-->
简单的说，就是有一个抽象类定义了一个方法，而实现类来决定到底初始化那个实例。这些实例，就是一个个产品了。   
产品接口（主要是考虑面向接口变成吧，个人感觉不要也行，对理解模式没有影响）：   

	public interface IProduct {
		public void sayName();
	}
下面呢，假设有两种产品：
	
	public class ProductA implements IProduct {
		@Override
		public void sayName() {
			// TODO Auto-generated method stub
			System.out.println("I am ProductA");
		}
	}
	public class ProductB implements IProduct {
		@Override
		public void sayName() {
			System.out.println("I am productB");
		}
	}
有了产品定义，那么下面就要建一个工厂了，怎么建呢？依据定义来说，首先要定义一个接口了：   

	public abstract class AbstractProductFactory {
		public abstract <T extends IProduct> T createProduct(
			Class<T> productType);
	}
下面就是一个子类了，也就是具体负责初始化实例的工厂了。

	public class ProductFactory extends AbstractProductFactory {

		@Override
		public <T extends IProduct> T createProduct(Class<T> productType) {
			// TODO Auto-generated method stub
			IProduct product = null;
			try {
				product = productType.newInstance();
			} catch (InstantiationException e) {
				e.printStackTrace();
			} catch (IllegalAccessException e) {
				e.printStackTrace();
			}
			return (T) product;
		}
	}
如何生成不同的产品呢，本例是根据传入的不同的类，来返回不同的实例的。当然了也可以用一个标识符了,如果传入的是“A”那么返回一个ProductA。诸如此类，这就是工厂方法了。  
具体使用时，是这样的：   

	ProductFactory productFactory = new ProductFactory();
	IProduct productA = productFactory.createProduct(ProductA.class);
	IProduct productB = productFactory.createProduct(ProductB.class);
	productA.sayName();
	productB.sayName();

##简单工厂##
简单工厂，可以说是工厂方法的一种扩展。上例中，发现在使用`ProductFactory`的时候，还需要先实例化一个。怎么那么麻烦呢？就像在实际生活中，我想要某个产品，我还需要先建一个工厂是一个道理的。   

还好，Java提供了这样一个关键字`static`,简单工厂类就变成这个样子了。

	public class SimpleProductFactory {

		public static <T extends IProduct> T createProduct(Class<T> productType) {
			// TODO Auto-generated method stub
			IProduct product = null;
			try {
				product = productType.newInstance();
			} catch (InstantiationException e) {
				e.printStackTrace();
			} catch (IllegalAccessException e) {
				e.printStackTrace();
			}
			return (T) product;
		}
	}
这样在使用的时间是方便了不少呢？

	IProduct productA = SimpleProductFactory.createProduct(ProductA.class);
	IProduct productB = SimpleProductFactory.createProduct(ProductB.class);
	productA.sayName();
	productB.sayName();
##抽象工厂##
情况进一步发展，大家对美的追求不断提高。工厂也是需要对自己的产品不断升级的。那就对现有的产品ProductA，ProductB进行升级，各自推出红，蓝两种颜色的产品。那么，我就需要两个工厂了，一个工厂来生产红色产品，一个工厂来生产蓝色产品。   
下面，先对产品进行改造：  

	public interface IProduct {
		public void sayName();
		public void sayColor();
	}
	public abstract class ProductA implements IProduct {
		@Override
		public void sayName() {
			// TODO Auto-generated method stub
			System.out.println("I am ProductA");
		}
	}
	public abstract class ProductB implements IProduct {
		@Override
		public void sayName() {
			System.out.println("I am productB");
		}
	}
哎呀，貌似ProductA，ProductB没什么变化啊！还是有些变化的，都变为`abstract`了，也就是说他俩都是半成品，还没给上色呢。怎么能实例化，然后往外销售呢，这不坑人，影响工厂形象啊。当然了，没上色，那也没办法`sayColor`了，鬼知道将会涂成什么颜色。   

好吧，有了半成品，现在进行上色操作。  

	public class RedProductA extends ProductA {
		@Override
		public void sayColor() {
			System.out.println("my color is Red!");
		}
	}
	public class BlueProductA extends ProductA {
		@Override
		public void sayColor() {
			System.out.println("my color is Blue!");
		}
	}
	
	public class RedProductB extends ProductB {
		@Override
		public void sayColor() {
			System.out.println("my color is Red!");
		}
	}
	public class BlueProductB extends ProductB {
		@Override
		public void sayColor() {
			System.out.println("my color is Blue!");
		}
	}
到此为止，产品定义完成了。    
下面就开建工厂了。一个**红色工厂**、一个**蓝色工厂**   
先来个抽象的：   
	
	public abstract class AbstractProductFactory {
		public abstract IProduct createProductA();

		public abstract IProduct createProductB();
	}
工厂就是为负责生产两种产品的了。

	public class RedProductFactory extends AbstractProductFactory {

		@Override
		public IProduct createProductA() {
			return new RedProductA();
		}

		@Override
		public IProduct createProductB() {
			return new RedProductB();
		}
	}

	public class BlueProductFactory  extends AbstractProductFactory{
		@Override
		public IProduct createProductA() {
			return new BlueProductA();
		}

		@Override
		public IProduct createProductB() {
			return new BlueProductB();
		}
	}
那么，通过这两种工厂生产出的产品，不论是A,还是B，颜色铁定是一致的啊。
使用场景如下：  

		RedProductFactory redProductFactory = new RedProductFactory();
		System.out.println("red factory is producing");
		IProduct product1 = redProductFactory.createProductA();
		product1.sayName();
		product1.sayColor();
		IProduct product2 = redProductFactory.createProductB();
		product2.sayName();
		product2.sayColor();
		
		BlueProductFactory blueProductFactory = new BlueProductFactory();
		System.out.println("blue factory is producing");
		IProduct product3 = blueProductFactory.createProductA();
		product3.sayName();
		product3.sayColor();
		IProduct product4 = blueProductFactory.createProductB();
		product4.sayName();
		product4.sayColor();

这样两个工厂分别开工了，一个出的产品都是红色的，一个都是蓝色的。这就是**抽象工厂**了。
##工厂模式 in JDK##


	Class.forName(String className);   

这种应该就是简单工厂的典型了。依据不同的className来生产相应的对象，只不过这里是Class对象了，不要混淆。

	javax.xml.parsers.DocumentBuilderFactory#newInstance()
	javax.xml.transform.TransformerFactory#newInstance()
	javax.xml.xpath.XPathFactory#newInstance()
这几个为什么就是抽象工厂了呢？不理解。高人指点。

---