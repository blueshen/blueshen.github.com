---
layout: post
title: "设计模式：观察者（Observer） in java"
date: 2012-11-08 18:50
comments: true
categories: 设计模式
tags: [ Java, Observer, Observable ]
---
**定义：**又叫发布订阅模式（Publish/subscribe），它定义了对象间的一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并自动更新。  
>这个定义还是比较通俗易懂的。我看了一遍，发现这不就是微博吗？我发布一条微博，那么所有关注我的人，都会收到通知，然后在新鲜事里显示出来。没错，就是这样！
	
观察者模式有4个角色：  

* 被观察者(Observable):  
定义被观察者必须实现的职责，动态的增加、删除观察者以及通知观察者   
* 观察者（Observer）:   
接收到消息后，进行更新操作
* 被观察者(Observable)具体类:  
定义自己的业务逻辑，并定义哪儿些事件需要通知观察者   
* 观察者（Observer）具体类：     
每个观察者在接收到消息后的更新操作是不同的。
 

<!--more-->
在Java中如何实现观察者模式呢？废话，写代码啊！这个我自然知道，更令人惊喜的是JDK自身就提供了对观察者模式的原生支持，我不得不赞叹Java的强大。  

Java提供了这样的两个东西：  
	
* 类`java.util.Observable`：  
它内部维护了一个Vector容器，用来放所有的观察者，并且提供了添加、删除观察者的方法。此外，定义了notifyObservers方法，用来通知观察者。  
* 接口`java.util.Observer`：
它定义了一个update方法，让Observer具体类来实现各自的操作。

下面，就以微博作为例子吧。  
先来一个被观察者，也就是我自己了。

	public class ShenYanChao extends Observable {

		//业务逻辑,不通知
		public String getName(){
			return "shenyanchao";
		}
		//发微博，通知
		public void publishWeibo(String content){
			System.out.println("我发布1条微博，內容是：["+content+"]");
			setChanged();
			notifyObservers(content);
		}
	}
其中，setChanged()用来表明自身的状态变了，否则观察者是不会理的。这个是JDK的限制，其实观察者模式可以不用这个的。  

下面就需要定义观察者了，也就是我的粉丝了。

	public class Fans implements Observer {

		@Override
		public void update(Observable o, Object content) {
			String who = ((ShenYanChao) o).getName();
			System.out.println("新鲜事:{" + who + "发布了一条微博，内容是：[" + content + "]}");
		}

	}
观察者Fans一旦发现我发了1条微博，那么他就会出现一条新鲜事的了。update()的参数，第1个是被观察者，也就是我；第2个就是notifyObservers传过来的参数了，此处是微博内容。

具体场景是这样的：

	ShenYanChao shenyanchao = new ShenYanChao();
	final int FANS_NUM = 10;//我的粉丝可不止这些呢
	for (int i = 0; i < FANS_NUM; i++) {
		shenyanchao.addObserver(new Fans());
	}
	shenyanchao.publishWeibo("欢迎登录：www.shenyanchao.cn");

此处模拟，我有10个粉丝，然后我发了1条微博。结果如下：  

	我发布1条微博，內容是：[欢迎登录：www.shenyanchao.cn]
	新鲜事:{shenyanchao发布了一条微博，内容是：[欢迎登录：www.shenyanchao.cn]}
	新鲜事:{shenyanchao发布了一条微博，内容是：[欢迎登录：www.shenyanchao.cn]}
	新鲜事:{shenyanchao发布了一条微博，内容是：[欢迎登录：www.shenyanchao.cn]}
	新鲜事:{shenyanchao发布了一条微博，内容是：[欢迎登录：www.shenyanchao.cn]}
	新鲜事:{shenyanchao发布了一条微博，内容是：[欢迎登录：www.shenyanchao.cn]}
	新鲜事:{shenyanchao发布了一条微博，内容是：[欢迎登录：www.shenyanchao.cn]}
	新鲜事:{shenyanchao发布了一条微博，内容是：[欢迎登录：www.shenyanchao.cn]}
	新鲜事:{shenyanchao发布了一条微博，内容是：[欢迎登录：www.shenyanchao.cn]}
	新鲜事:{shenyanchao发布了一条微博，内容是：[欢迎登录：www.shenyanchao.cn]}
	新鲜事:{shenyanchao发布了一条微博，内容是：[欢迎登录：www.shenyanchao.cn]}
可见，一旦我发了微博，所有的观察者（Fans）都收到了。

