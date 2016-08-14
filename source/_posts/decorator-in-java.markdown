---
layout: post
title: "设计模式：装饰器（Decorator）in Java"
date: 2012-10-30 21:25
comments: true
categories: 设计模式
tags: [ Java, decorator, IO ]
---
1.认识装饰器模式

装饰模式能够实现动态的为对象添加功能，是从一个对象外部来给对象添加功能。通常给对象添加功能，要么直接修改对象添加相应的功能，要么派生对应的子类来扩展，抑或是使用对象组合的方式。显然，直接修改对应的类这种方式并不可取。在面向对象的设计中，而我们也应该尽量使用对象组合，而不是对象继承来扩展和复用功能。装饰器模式就是基于对象组合的方式，可以很灵活的给对象添加所需要的功能。装饰器模式的本质就是动态组合。动态是手段，组合才是目的。总之，装饰模式是通过把复杂的功能简单化，分散化，然后再运行期间，根据需要来动态组合的这样一个模式。 
 
<!--more-->
2.模式结构和说明  

装饰模式的结构如下图所示。

Component：组件对象的接口，可以给这些对象动态的添加职责；

ConcreteComponent：具体的组件对象，实现了组件接口。该对象通常就是被装饰器装饰的原始对象，可以给这个对象添加职责；

Decorator：所有装饰器的父类，需要定义一个与组件接口一致的接口(主要是为了实现装饰器功能的复用，即具体的装饰器A可以装饰另外一个具体的装饰器B，因为装饰器类也是一个Component)，并持有一个Component对象，该对象其实就是被装饰的对象。如果不继承组件接口类，则只能为某个组件添加单一的功能，即装饰器对象不能在装饰其他的装饰器对象。

ConcreteDecorator：具体的装饰器类，实现具体要向被装饰对象添加的功能。用来装饰具体的组件对象或者另外一个具体的装饰器对象。

装饰器模式的示例代码如下(Java语言描述)：

   (1)组件对象的接口，可以给这些对象动态的添加职责

	public abstract class Component {  
    	public abstract void operation();  
	}  
(2)具体实现组件对象接口的对象
  
	public class ConcreteComponent extends Component {  
  
    	public void operation() {  
        	//相应的功能处理  
    	}  
  	}  
 
(3)装饰器接口，维持一个指向组件对象的接口对象， 并定义一个与组件接口一致的接口
 
	public abstract class Decorator extends Component {  
    	/** 
     	* 持有组件对象 
     	*/  
    	protected Component component;  
  
    	/** 
     	* 构造方法，传入组件对象 
     	* @param component 组件对象 
    	*/  
    	public Decorator(Component component) {  
        	this.component = component;  
    	}  
  
    	public void operation() {  
        	//转发请求给组件对象，可以在转发前后执行一些附加动作  
        	component.operation();  
    	}   
	}  
 

(4)装饰器的具体实现对象，向组件对象添加职责，operationFirst()，operationLast()为前后需要添加的功能。具体的装饰器类ConcreteDecoratorB代码相似，不在给出。
 
	public class ConcreteDecoratorA extends Decorator {  
       	public ConcreteDecoratorA(Component component) {  
            super(component);  
   		}  
       private void operationFirst(){ } //在调用父类的operation方法之前需要执行的操作  
       private void operationLast(){ } //在调用父类的operation方法之后需要执行的操作  
       public void operation() {  
           //调用父类的方法，可以在调用前后执行一些附加动作  
           operationFirst(); //添加的功能  
           super.operation();  //这里可以选择性的调用父类的方法，如果不调用则相当于完全改写了方法，实现了新的功能  
           operationLast(); //添加的功能  
   		}  
	}  
(5) 客户端使用装饰器的代码

	public class Client{  
   		public static void main(String[] args){  
    	Component c1 = new ConcreteComponent (); //首先创建需要被装饰的原始对象(即要被装饰的对象)  
    	Decorator decoratorA = new ConcreteDecoratorA(c1); //给对象透明的增加功能A并调用  
    	decoratorA .operation();  
    	Decorator decoratorB = new ConcreteDecoratorB(c1); //给对象透明的增加功能B并调用  
    	decoratorB .operation();  
    	Decorator decoratorBandA = new ConcreteDecoratorB(decoratorA);//装饰器也可以装饰具体的装饰对象，此时相当于给对象在增加A的功能基础上在添加功能B  
    	decoratorBandA.operation();  
  		}  
	}  
 
3.小结

Java中的IO是明显的装饰器模式的运用。FilterInputStream，FilterOutputStream，FilterRead，FilterWriter分别为具体装饰器的父类，相当于Decorator类，它们分别实现了InputStream，OutputStream，Reader，Writer类(这些类相当于Component，是其他组件类的父类，也是Decorator类的父类)。继承自InputStream，OutputStream，Reader，Writer这四个类的其他类是具体的组件类，每个都有相应的功能，相当于ConcreteComponent类。而继承自FilterInputStream，FilterOutputStream，FilterRead，FilterWriter这四个类的其他类就是具体的装饰器对象类，即ConcreteDecorator类。通过这些装饰器类，可以给我们提供更加具体的有用的功能。如FileInputStream是InputStream的一个子类，从文件中读取数据流，BufferedInputStream是继承自FilterInputStream的具体的装饰器类，该类提供一个内存的缓冲区类保存输入流中的数据。我们使用如下的代码来使用BufferedInputStream装饰FileInputStream，就可以提供一个内存缓冲区来保存从文件中读取的输入流。

BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file)); //其中file为某个具体文件的File或者FileDescription对象  
    在以下两种情况下可以考虑使用装饰器模式：

    (1)需要在不影响其他对象的情况下，以动态、透明的方式给对象添加职责。

    (2)如果不适合使用子类来进行扩展的时候，可以考虑使用装饰器模式。


参考文档：<http://www.cnblogs.com/chenying99/archive/2012/10/05/2712524.html>