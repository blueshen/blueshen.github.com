---
layout: post
title: "设计模式：适配器（Adapter）in Java"
date: 2012-10-30 21:24
comments: true
categories: 设计模式
tags: [ 设计模式, Java, Adapter ]
---
**类适配器**:客户的开发人员定义了一个接口，期望用这个接口来完成整数的求和操作，接口定义如下：     
Java代码    

	public interface Operation{  
      	public int add(int a,int b);  
	}  

开发人员在了解这个接口的定义后，发现一个第三方类，里面有一个方法能实现他们期望的功能，其代码如下：    
Java代码    

	public class OtherOperation{  
      	public int otherAdd(int a,int b){  
           	return a + b;  
      	}  
	}  
<!--more-->
以上第三方类OtherOperation的方法public int otherAdd(int a,int b)所提供的功能，完全能符合客户的期望，所以只需要想办法把OtherOperation的otherAdd(int a,int b)和客户的Operation接口联系起来，让这个第三方类来为客户提供他们期望的服务就行了，这样就避免了开发人员再度去研究类似OtherOperation的otherAdd(int a,int b)方法的实现（利用已有的轮子，避免重复发明），这方法之一，就是用适配器模式：   
Java代码  

	public class AdapterOperation extends OtherOperation implements Operation{  
      	public int add(int a,int b){  
           return otherAdd(a,b);  
     	 }  
	}  

以上就是适配器的实现方法之一，**类适配器**，在以上实现中存在着三中角色分别是：    
1：适配目标角色：Operation。  
2：适配类（原）角色：OtherOperation。  
3：适配器角色：AdapterOperation。  
其中适配器角色是适配器模式的核心。 
适配器的主要工作就是通过封装现有的功能，使他满足需要的接口。 

**对象适配器**  
我们再来看看另一种情况： 
假如客户接口期望的功能不止一个，而是多个：   
Java代码   

	public interface Operation{  
      	public int add(int a,int b);  
      	public int minus(int a,int b);  
      	public int multiplied(int a,int b);  
	}  

而能提供这些实现的原可能不止一个：   
Java代码   

	public class OtherAdd{  
      	public int otherAdd(int a,int b){  
          	 return a + b;  
      	}  
	}  
  
	public class OtherMinus{  
      	public int minus(int a,int b){  
           return a - b;  
      	}  
	}  
  
	public class OtherMultiplied{  
      	public int multiplied(int a,int b){  
           return a * b;  
      	}  
	}  

由于java是不能实现多继承的，所以我们不能通过构建一个适配器，让他来继承所有原以完成我们的期望，这时候怎么办呢?只能用适配器的另一种实现--对象适配器：  
Java代码   

	public class AdapterOperation implements Operation{  
      	private OtherAdd add;  
      	private OtherMinus minus;  
      	private OtherMultiplied multiplied;  
  
      	public void setAdd(OtherAdd add){  
            this.add = add;  
      	}  
  
      	public void setMinus(OtherMinus minus){  
            this.minus = minus;  
      	}  
  
      	public void setMultiplied(OtherMultiplied multiplied){  
            this.multiplied = multiplied;  
      	}  
  
      	//适配加法运算  
      	public int add(int a,int b){  
           return add.otherAdd(a,b);  
      	}  
  
      	//适配减法运算  
      	public int minus(int a,int b){  
          return minus.minus(a,b);  
      	}  
  
      	//适配乘法运算  
     	public int multiplied(int a,int b){  
         return multiplied.multiplied(a,b);  
      	}  
	}  

上面代码很明显，适配器并不是通过继承来获取适配类（原）的功能的，而是通过适配类的对象来获取的，这就解决了java不能多继承所带来的不便了。这也是java提倡的编程思想之一，即尽量使用聚合不要使用继承。  
还有一种情况是需要使用对象适配器的。我们来看看， 
单我们的客户提供的需求并不是一个明确的接口，而是一个类，并没有定义期望的方法，如下   
Java代码   
 
	public class A{  
   		public int add(int a,int b){  
      		return a + b;  
   		}  
	}  

现在客户要一个新类B，要求能在保留类A功能的情况下增加一个运算减法的功能，并要求B能随时替换掉A但不能对已有系统造成影响。这样我们只能新建一个类B，并让B继承A。 	
Java代码  

	public class B extends A{  
    	B(){  
      		super();  
    		}  
  
    	public int minus(int a,int b){  
           //待实现的减法运算函数..  
    	}  
	}  

这时候，我们发现类C已经提供了实现减法的函数，		 
Java代码   
  
	public class C{  
    	public int minus(int a,int b){  
           return a - b;  
    	}  
	}  

为了避免重复去设计该函数，我们决定引入C类，通过适配C类来达到我们的期望，但问题是A和C都是一个具体类，我们无法让B同时继承这个两个类，而B继承A又是必须的，所以我们只能考虑把C给内聚到B内部，对象适配器又得派上用场了。  
Java代码  	

	public class B extends A{  
    	private C c;  
  
    	B(){  
      		super();  
    		}  
  
   		public void setMinus(C c){  
         	this.c= c;  
    	}  
  
    	public int minus(int a,int b){  
           return c.minus(a,b);  
    	}  
	}  
这样，在需要A类的地方都能用B类来代替，同时又保证了新的功能的引入。 	

更灵活的实现--**隐藏目标接口的抽象适配器**

做java 桌面应用的都知道WindowListener接口，   
Java代码  

	public interface WindowListener extends EventListener{  
		 public void windowActivated(WindowEvent e)；  
		 public void windowClosed(WindowEvent e)；  
		 public void windowClosing(WindowEvent e)；  
		 public void windowDeactivated(WindowEvent e)；  
		 public void windowDeiconified(WindowEvent e)；  
		 public void windowIconified(WindowEvent e)；  
		 public void windowOpened(WindowEvent e)；  
	}  
要实现这个接口，我们就必须实现它所定义的所有方法，但是实际上，我们很少需要同时用到所有的方法，我们要的只是其中的两三个。为了不使我们实现多余的方法，		 
jdk WindowListener提供了一个WindowListener的默认实现类WindowAdapter类，这是一个抽象类，   
Java代码		
  
	public abstract class WindowAdapter implements WindowListener{  
 		public void windowActivated(WindowEvent e){}  
 		public void windowClosed(WindowEvent e){}  
 		public void windowClosing(WindowEvent e){}  
 		public void windowDeactivated(WindowEvent e){}  
		public void windowDeiconified(WindowEvent e){}  
 		public void windowIconified(WindowEvent e){}  
 		public void windowOpened(WindowEvent e){}  
	}  

WindowAdapter类对WindowListener接口的所有有方法都提供了空实现， 
有了WindowAdapter类，我们只需要去继承WindowAdapter，然后选择我们所关心的方法来实现就行了，这样就避免了直接去实现WindowListener接口。   

参考文献：<http://chjl2020.iteye.com/blog/262370>