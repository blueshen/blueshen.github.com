---
layout: post
title: "设计模式：策略（strategy） in java"
date: 2012-11-02 20:11
comments: true
categories: 设计模式
tags: [ strategy, policy, pattern, 设计模式 ]
---
**定义：**策略模式（Strategy Pattern）是一种比较简单的模式，也叫做政策模式（Policy Pattern）。定义一组算法，将每个算法都封装起来，并且使他们之间可以互换。    

策略模式有3个角色：   

- Context封装角色    
它叫上下文角色，起承上启下封装的作用，屏蔽高层模块对策略、算法的直接访问，封装可能存在的变化。
- Strategy抽象策略角色    
策略、算法的家族的抽象，通常为接口。   
- ConcreteStrategy具体策略角色    
实现抽象策略中的操作，该类含有具体的算法。  

下面借用刘备江东娶亲，诸葛亮给赵云3个妙计的故事来说明这个问题。    
<!--more-->
这3个妙计，分别是：   

- 找乔国老帮忙（走后门）
- 求吴国太放行（诉苦）
- 孙尚香断后    

首先，设计一个妙计的策略接口：    

    public interface IStrategy{
        public void operate();
    }    

下面来分别实现这几个妙计。   

    //妙计1
    public class BackDoor implements IStrategy{    
        public void operate(){
            System.out.println("找乔国老帮忙，让吴国太给孙权施加压力");
        }
    }
    //妙计2
    public class GivenGreenLight implements IStrategy{    
        public void operate(){
            System.out.println("找吴国太开绿灯，放行！");
        }
    }
    //妙计3
    public class BlockEnemy implements IStrategy{    
        public void operate(){
            System.out.println("孙尚香断口，挡住追兵！");
        }
    }

这几个妙计（算法）都写好了。那么如何使得他们之间可以互换呢。这就需要使用Context进行封装了。在本例子中锦囊就是这个作用，它承载了三种策略妙计。    

    public class Context{
        private IStrategy strategy;
        public Context(IStrategy strategy){
            this.strategy = strategy;
        }
        
        public void operate(){
            this.strategy.operate();
        }
    }
通过构造函数把策略传递进来，实现了不同策略的互换，同时提供了统一的operate()方法让高层（赵云）使用。    

下面看下赵云如何使用的。    

    public class ZhaoYun{   
        Context context;
        System.out.println("---刚到吴国的时候拆第一个---");
        context = new Context(new BackDoor());
        context.operate();
        System.out.println("---刘备乐不思蜀了，拆第二个---");
        context = new Context(new GivenGreenLight());
        context.operate();
        System.out.println("---孙权的小兵过来追，拆第三个---");
        context = new Context(new BlockEnemy());
        context.operate();
    }


​    
​    
需要注意的是，策略方法提供了各种策略的互换，以及高层调用。但是具体什么条件下，使用什么策略是需要外部来判断的。本例子只是按顺序执行了。     

​    