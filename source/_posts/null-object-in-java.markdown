---
layout: post
title: "设计模式：空对象（null object） in java"
date: 2012-11-02 20:10
comments: true
categories: 设计模式  
tags: [ null, object, 设计模式 ]
---
相信大家一定在开发中见过并且写过类似这样的代码：    

```java
public Book getBook(int id) {  
    if (id < 0) {  
        return null;  
    }  
    return new Book(1, "Design Pattern", 100);  
}  

Book book = getBook(-1);  
if (book != null) {                   
}
```

系统在使用对象的相关功能时，总要检查对象是否为null，如果不为null，我们才会调用它的相关方法，完成某种逻辑。这样的检查在一个系统中出现很多次，相信任何一个设计者都不愿意看到这样的情况。为了解决这种问题，我们可以可以引入空对象，这样，我们就可以摆脱大量程式化的代码，对代码的可读性也是一个飞跃。    

<!--more--> 

等等，空对象是什么？它和null什么关系？    
空对象是一个没有实质性内容的对象，但他并不为null。你可以把空对象理解为一个空箱子，这个物品还是存在的，只不过仅仅是一个壳，没有实质性的东西。    
我们需要对原有的代码进行重构，把所有返回null的地方都替换成返回一个与之对应的空对象，然后再客户端调用时不再使用book==null这种方式判断是否为空，而是替换成book.isNull()的方式。下面我们就一步一步来实现这种模式。    

首先，我们需要定义一个Nullable接口：   

```java
public interface Nullable {  
    /** 
     * 对象是否为空 
     * @return 
     */  
    public boolean isNull();  
}  
```
这个接口定义了一个isNull()的方法，来表示一个对象是否为空。   
然后我们让Book实现此接口：    

```java
public class Book implements Nullable {  
  
    private int id;  
    private String name;  
    private double price;  
      
    public Book() {  
    }  
      
    public Book(int id, String name, double price) {  
        this.id = id;  
        this.name = name;  
        this.price = price;  
    }  
      
    @Override  
    public boolean isNull() {  
        return false;  
    }  
  
    /** 
     * 创建一个NullBook实例代表空对象 
     * @return 
     */  
    public static Book createNullBook() {  
        return new NullBook();  
    }  
      
    /** 
     * setters & getters 
     */  
}
```

在Book中实现了isNull()方法，并返回false；另外Book还定义了一个静态的createNullBook方法，返回了一个NullBook的实例，NullBook是什么呢，我们来看一下：   

```java
public class NullBook extends Book {  
    @Override  
    public boolean isNull() {  
        return true;  
    }  
}  
```

NullBook继承了Book，并在isNull()方法中返回true，这个NullBook其实就是我们上面提到的空对象，仅仅是个空箱子而已。    
然后我们定义一个BookService类，用以模拟获取Book的接口方法：    

```java
public class BookService {  
    public Book getBook(int id) {  
        if (id < 0) {  
            //返回一个空对象  
            return Book.createNullBook();             
        }  
        return new Book(id, "Design Pattern", 100);  
    }  
}  
```

在getBook(int id)中，如果`id<0`时，则返回一个NullBook的实例，在客户端调用isNull()方法时则返回true，表示是空对象；如果`id>=0`时，则返回一个Book的实例，在客户端调用isNull()方法时则返回false，表示对象不为空，可以调用相关方法取得数据。对于我们的客户端来讲，返回的都是一个Book类型的对象，我们并不清楚返回的到底是真正的Book实例还是空对象NullBook实例，但是我们并不担心，因为有一点可以肯定的是，我们都可以放心的调用isNull()方法，因为它会根据运行期对象的类型返回一个正确的值。   

多态的最根本好处在于：你不必再向对象询问“你是什么类型”而后根据得到的答案调用对象的某个行为，你只管调用该行为就是了，其他的一切事情多态机制会为你妥善处理。    

最后再来看一下客户端是如何实现对象为空的判断的：    

```java
public class Client {  
    public static void main(String[] args) {  
        BookService service = new BookService();  
        Book book = service.getBook(-1);  
        if (book.isNull()) {  
            System.out.println("not found!");  
        } else {  
            System.out.println("name:" + book.getName());  
            System.out.println("price:" + book.getPrice());  
        }  
    }  
}  
```

---
原文：<http://blog.csdn.net/liuhe688/article/details/6586458>