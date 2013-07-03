---
layout: post
title: "设计模式：桥接(bridge) in java"
date: 2012-11-02 20:06
comments: true
categories: 设计模式
---
**定义：**Bridge 模式又叫做桥接模式，是构造型的设计模式之一。Bridge模式基于类的最小设计原则，通过使用封装，聚合以及继承等行为来让不同的类承担不同的责任。它的主要特点是把抽象（abstraction）与行为实现（implementation）分离开来，从而可以保持各部分的独立性以及应对它们的功能扩展。     

为什么要使用桥接模式？    


桥接模式的角色和职责  

- Client   
    Bridge模式的使用者
- Abstraction   
   它的主要职责是定义出该角色的行为，同时保存一个对实现化角色的引用，该角色一般是抽象类。   
- Refined Abstraction    
    修正抽象化角色。它引用实现化角色对抽象化角色进行修正。   
- Implementor    
    实现化角色。它是接口或者抽象类，定义角色必须的行为和属性。   
- ConcreteImplementor    
    它实现接口或者抽象类定义的方法和属性。     
    
