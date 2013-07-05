---
layout: post
title: "设计模式：桥接(bridge) in java"
date: 2012-11-02 20:06
comments: true
categories: 设计模式
tags: [ bridge, pattern ]
---
**定义：**Bridge 模式又叫做桥接模式，是构造型的设计模式之一。Bridge模式基于类的最小设计原则，通过使用封装，聚合以及继承等行为来让不同的类承担不同的责任。它的主要特点是把抽象（abstraction）与行为实现（implementation）分离开来，从而可以保持各部分的独立性以及应对它们的功能扩展。     

###为什么要使用桥接模式？    
场景：我们想绘制矩形、圆形、椭圆形、正方形，我们至少需要4个形状类。但是如果又需要绘制的图形是不同颜色的，比如白色、灰色、蓝色的。    
我们可能很快就会想到这样的方案：    

![方案1](/images/blog/bridge-pattern-1.png)        
按照上面的说法，我们可能要新建4*3=12个类来完成。   
但是如果需要画更多的图形，并有更多的颜色呢。如此扩展下去很可能出现类爆炸。   
<!--more-->
那如何解决呢？使用Bridge来组合这些方案吧。这种方案只需要4+3个类就搞定了。  
 
![方案2](/images/blog/bridge-pattern-2.png)

###桥接模式的角色和职责  

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
    
###使用Bridge模式来实现绘图场景
Abstraction：    

    public abstract class Shape {
    
        protected   Color color;
        public Shape(){
            this.color = new BlackColor();
        }
        public Shape(Color color){
            this.color = color;
        }
    
        public void setColor(Color color) {
            this.color = color;
        }
        public abstract void draw();
    
    }
Refined Abstraction：   

    //圆形
    public class CircleShape extends Shape{
    
        public CircleShape(){
            super();
        }
        public CircleShape(Color color){
            super(color);
        }
    
        @Override
        public void draw() {
            System.out.println("画一个圆形");
            color.draw();
        }
    }
    //椭圆形
    public class EllipseShape extends Shape {
    
        public EllipseShape(){
            super();
        }
        public EllipseShape(Color color){
            super(color);
        }
    
        @Override
        public void draw() {
            System.out.println("画一个椭圆形");
            color.draw();
        }
    }
    //矩形
    public class RectangleShape extends Shape {
    
        public  RectangleShape(){
            super();
        }
        public  RectangleShape(Color color){
            super(color);
        }
    
        @Override
        public void draw() {
            System.out.println("画一个矩形");
            color.draw();
        }
    }
    
Implementor：   

    public interface Color {
    
        public void draw();
    
    }
ConcreteImplementor：   
 
    public class WhiteColor implements Color {
        @Override
        public void draw() {
            System.out.println("颜色是白色的");
        }
    }  
    
    public class GrayColor implements Color {
    
        @Override
        public void draw() {
            System.out.println("颜色是灰色的");
        }
    }
    
    public class BlackColor implements Color {
    
        @Override
        public void draw() {
            System.out.println("颜色是黑色的");
        }
    } 
    
上面已经完成了各个部分，接下来就组合来使用这些类画出不同颜色的各种图形了。     

    public class Client {
    
        public static void main(String[] args) {
            Color whiteColor = new WhiteColor();
            Color grayColor = new GrayColor();
            Color blackColor = new BlackColor();
    
            Shape circleShape = new CircleShape();
            Shape ellipseShape = new EllipseShape();
            Shape rectangleShape = new RectangleShape();
    
            //画一个黑色的圆形
            circleShape.setColor(blackColor);
            circleShape.draw();
    
            //画一个白色的椭圆
            ellipseShape.setColor(whiteColor);
            ellipseShape.draw();
    
            //画一个灰色的矩形
            rectangleShape.setColor(grayColor);
            rectangleShape.draw();
        }
    }
这样就实现了。各种组合来满足条件。如果要添加更多的颜色以及图形，只需要分别扩展就行，不用该原来的代码。使用的时候随心组合就OK了。    
