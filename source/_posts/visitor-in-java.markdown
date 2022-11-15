---
layout: post
title: "设计模式：访问者（visitor） in java"
date: 2012-11-02 20:12
comments: true
categories: 设计模式
tags: [ visitor, pattern, 访问者, 设计模式 ]
---
**定义：**封装一些作用于某种数据结构中的各元素的操作，它可以在不改变数据结构的前提下定义作用于这些元素的新的操作。

访问者模式有以下几个角色：

- Visitor抽象访问者 
抽象类或者接口，声明访问者可以访问哪些元素，具体到程序中就visit方法的参数定义哪些对象是可以被访问的。
- ConcreteVisitor具体访问者 
它影响访问者访问到一个类后该怎么办，要做什么事情。
- Element抽象元素 
接口或者抽象类，声明接受哪一类访问者访问。程序上是通过accept方法中的参数来定义的。
- ConcreteElement具体元素
实现accept方法，通常是visitor.visit(this),基本都形成一个模式了。
- ObjectStruture结构对象
元素产生者，一般容纳在多个不同类，不同接口的容器。项目中，一般很少抽象这个角色。   

<!--more-->
下面看看各个部分是如何实现的。 

抽象元素：

```java
public abstract class Element {
    //业务逻辑
    public abstract void doSomething();
    //允许谁来访问
    public abstract void accept(IVisitor visitor);
}
```
具体元素：

```java
public class ConcreteElement1 extends Element {

    @Override
    public void doSomething() {
        //todo
    }

    @Override
    public void accept(IVisitor visitor) {
        visitor.visit(this);
    }
}

public class ConcreteElement2 extends Element {

    @Override
    public void doSomething() {
        //todo
    }

    @Override
    public void accept(IVisitor visitor) {
        visitor.visit(this);
    }
}  
```

抽象访问者：

```java
public interface IVisitor {

    public void visit(ConcreteElement1 el1);

    public void visit(ConcreteElement2 el2);
}
```
具体访问者：

```java
public class Visitor implements IVisitor {

    @Override
    public void visit(ConcreteElement1 el1) {
        el1.doSomething();
    }

    @Override
    public void visit(ConcreteElement2 el2) {
        el2.doSomething();
    }
}
```

结构对象：

```java
public class ObjectStruture {

    public static Element createElment(){
        Random random = new Random();
        if (random.nextInt(100) > 50){
            return new ConcreteElement1();
        }else{
            return new ConcreteElement2();
        }
    }

}
```

下面看下具体场景类是怎么使用的： 

```java
public class Client {

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Element el = ObjectStruture.createElment();
            el.accept(new Visitor());
        }
    }
}
```
