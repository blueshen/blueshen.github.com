---
layout: post
title: "设计模式：组合（composite） in java"
date: 2012-11-02 20:07
comments: true
categories: 设计模式
tags: [ composite, pattern ]
---
**定义：**组合模式（Composite Pattern）也叫合成模式，有时又叫做部分-整体模式（Part-Whole），主要是用来描述部分与整体的关系。确切的定义：将对象组合成树形结构以表示“部分-整体”的层次结构，使得用户对单个对象和组合对象的使用具有一致性。   

组合模式类图如下：    

![组合模式](/images/blog/composite-pattern.jpg)    

组合模式有以下几个角色：   

- Component抽象构件角色   
定义参加组合对象的共有方法和树形，可以定义一些默认的行为或属性。   
- Leaf叶子构件    
叶子对象，下面没有其他分支，也就是最小的遍历单位。   
- Composite树枝构件    
树枝对象，它的作用是组合树枝节点和叶子节点形成一个树形结构。    


这个说白了就是一个树形结构，不再具体使用例子了。     
