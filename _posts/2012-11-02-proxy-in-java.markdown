---
layout: post
title: "设计模式：代理（Proxy） in java"
date: 2012-11-02 16:21
comments: true
categories: 设计模式
tags: [ Java, proxy, JDK, cglib ]
---
##什么是代理？##
代理是指，本该有A做的工作，现在找一个代理人B，然后由B来进行实际的工作。  
代理，简单来分，可以分为以下两类：   

* 静态代理
* 动态代理
<!--more-->

##静态代理##
以实现两个数的加法场景为例：   

	public interface IAdd {
		public int add(int a, int b);
	}
实现类：

	public class Add implements IAdd {
		@Override
		public int add(int a, int b) {
			return a + b;
		}
	}
直接使用的话：
	
	Add add = new Add();
	add.add(3, 14);
那么我想在执行加运算时，做一些其他操作怎么办，已有的类ADD无法改，没有源码。这时很容易想到的就是扩展：

	public class AddProxy implements IAdd {
		private IAdd add;

		public AddProxy(IAdd add) {
			this.add = add;
		}
		@Override
		public int add(int a, int b) {
			System.out.println("...begin...");
			int result = add.add(3, 14);
			System.out.println("...end...");
			return result;
		}
	}
这样做，没有修改已有的类，并且增加了一些操作，此处为一些提示信息。采用了组合的方式，实现了代理模式。具体使用时，直接使用AddProxy即可。   

	IAdd add = new AddProxy(new Add());
	int result = add.add(3, 14);
此为**静态代理**也。   
##动态代理##
动态代理，是指运行时动态的生成代理类，完成功能。静态代理中，显然AddProxy是编译期已知的了。实现方式，主要有两种：   

* JDK Proxy
* Cglib Proxy    

###JDK Proxy###
Java自身提供了相关的类，来实现动态代理。    
首先要定义一个`java.lang.reflect.InvocationHandler`接口实现   

	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	/**
 	* @author shenyanchao
 	*/
	public class AddInvocationHandler implements InvocationHandler {
		
		private Object target;
		//绑定要代理的目标类
		public void bind(Object target) {
			this.target = target;
		}

		@Override
		public Object invoke(Object proxy, Method method, Object[] args)
			throws Throwable {
			System.out.println("......begin....");
			Object result = method.invoke(target, args);
			System.out.println("......end....");
			return result;
		}
	}
那么在具体使用时，代码如下：  
	
	AddInvocationHandler addHandler = new AddInvocationHandler();
	IAdd add = new Add();
	addHandler.bind(add);
	IAdd addProxy = (IAdd) Proxy.newProxyInstance(
				Add.class.getClassLoader(), Add.class.getInterfaces(),
				addHandler);
	int jdkResult = addProxy.add(3, 14);
从代码可见，主要是通过`Proxy.newProxyInstance`来在运行时生成代理类。需要注意的是，第二个参数必须使用具体实现类Add来获得interfaces，也就是说其代理的类必须实现了接口。`addHandler`负责绑定要代理的target类，并调用invoke来增强Add功能。  
###Cglib Proxy###
JDK的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现JDK的动态代理，cglib是针对类来实现代理的，他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对final修饰的类进行代理。   
要使用CgLib，首先要实现一个CallBack接口的类，由于本例是为了实现method的拦截，因此直接实现MethodInterceptor即可：  

	import java.lang.reflect.Method;
	import net.sf.cglib.proxy.MethodInterceptor;
	import net.sf.cglib.proxy.MethodProxy;
	/**
 	* @author shenyanchao
 	*/
	public class AddInterceptor implements MethodInterceptor {

	@Override
	public Object intercept(Object obj, Method method, Object[] args,
			MethodProxy proxy) throws Throwable {
			System.out.println("....begin....");
			Object result = proxy.invokeSuper(obj, args);
			System.out.println("....end....");
			return result;
		}
	}
具体使用时：  

	Enhancer enhancer = new Enhancer();
	enhancer.setSuperclass(Add.class);
	enhancer.setCallback(new AddInterceptor());
	Add add  =  (Add) enhancer.create();
	int result = add.add(3, 14);
通过Enhancer制定需要增强的类，并设置CallBack函数来实现代理与功能增强。

