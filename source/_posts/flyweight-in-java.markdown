---
layout: post
title: "设计模式：享元（flyweight） in java"
date: 2012-11-02 20:07
comments: true
categories: 设计模式
tags: [ flyweight, pattern, 享元 ]
---
**定义：**享元模式（Flyweight Pattern）是池技术的重要实现方式。使用共享对象可有效地支持大量的细粒度的对象。    

享元模式提出了2个要求：细粒度的对象和共享对象。    
要求细粒度对象，那么不可避免地使用对象数量多且性质相近。可以将这些对象分为2个部分：    

- 内部状态（intrinsic）    
内部状态是对象可共享出来的信息，存储在享元对象内部并且不会随环境改变而改变。它们可以作为一个对象的动态附加信息，不必直接储存在具体的某个对象中，属于可以共享的部分。    
- 外部状态（extrinsic）   
外部状态是对象得以依赖的一个标记，是随环境改变而改变的，不可以共享的状态。    

<!--more-->
享元模式有以下几个角色：    

- Flyweight抽象享元角色
它简单说就是一个产品的抽象类，同时定义出对象的外部状态和内部状态的接口或实现。
- ConcreteFlyweight具体享元角色     
具体的一个产品类，实现抽象角色定义的业务。
- unsharedConcreteFlyweight不可共享的享元角色    
不存在外部状态或者安全要求不能够使用共享技术
- FlyweightFactory享元工厂    
职责非常简单，就是构造一个池容器，同时提供从池中获得对象的方法。    

抽象享元角色：   

```java
public abstract class Flyweight{
    //内部状态
    private String intrinstic;
    //外部状态
    protected final String Extrinsic;
    //要求享元角色必须接受外部状态
    public Flyweight(String _Extrinsic){
        this.Extrinsic = _Extrinsic;
    }
    //定义业务操作
    public abstract void operate();
    
    //内部状态的getter/setter
    public String getIntrinsic(){
        return intrinsic;
    }
    public void setIntrinsic(String intrinsic){
        this.intrinsic = intrinsic
    }
}
```

具体享元角色：   

```java
public class ConcreteFlyweight1 extends Flyweight{
    //接受外部状态
    public ConcreteFlyweight1(String _Extrinsic){
        super(_Extrinsic);
    }
    //根据外部状态进行逻辑处理
    public void operate(){
        //todo
    }    
}

public class ConcreteFlyweight2 extends Flyweight{
    //接受外部状态
    public ConcreteFlyweight1(String _Extrinsic){
        super(_Extrinsic);
    }
    //根据外部状态进行逻辑处理
    public void operate(){
        //todo
    }    
}
```

享元工厂：    

```java
public class FlyweightFactory{
    //定义一个池容器
    private static HashMap<String,Flyweight> pool = new HashMap<>(String,Flyweight);
    //享元工厂
    public static Flyweight getFlyweight(String Extrinsic){
        //需要返回的对象
        Flyweight flyweight = null;
        //在池中没有该对象
        if(pool.containsKey(Extrinsic)){
            flyweight = pool.get(Extrinsic);
        }else{
            //根据外部状态创建享元对象
            flyweight = new ConcreteFlyweight1(Extrinsic);
            //放置到池中
            pool.put(Extrinsic,flyweight);
        }
        return flyweight;
    }
    
}
```

享元模式的使用场景：   
（1）系统中存在大量的相似对象     
（2）细粒度的对象都具备较接近的外部状态，而且内部状态与环境无关，也就是说对象没有特定身份。       
（3）需要缓冲池的场景。    