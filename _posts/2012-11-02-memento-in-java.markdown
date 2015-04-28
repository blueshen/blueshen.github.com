---
layout: post
title: "设计模式:备忘录（memento） in java"
date: 2012-11-02 20:10
comments: true
categories: 设计模式
tags: [ memento, pattern, 备忘录 ]
---
**定义：**在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。   

备忘录模式有以下三个角色：   

- Originator发起人角色    
记录当前时刻的内部状态，负责定义哪些属于备份范围的状态，负责创建和恢复备忘录数据。   
- Memento备忘录角色     
负责存储Originator发起人对象的内部状态，在需要的时候提供发起人需要的内部状态。   
- Caretaker备忘录管理员角色    
对备忘录进行管理、保存和提供备忘录。   

<!--more-->
发起人角色：    

    public class Originator{
        //内部状态
        private String state = "";
        
        public String getState(){
            return state;
        }
        
        public void setState(String state){
            this.state = state;
        }
        
        //创建一个备忘录
        public Memento createMemento(){
            return new Memento(this.state);
        }
        //恢复一个备忘录
        public void restoreMemento(Memento _memento){
            this.setState(_memento.getState());
        }
    }
    
备忘录角色：    

    public class Memento{
        //发起人的内部状态
        private String state = "";
        //构造函数传递参数
        public Memento(String _state){
            this.state = _state;
        }  
        public String getState(){
            return state;
        }
        
        public void setState(String state){
            this.state = state;
        }
    }
    
备忘录管理员角色：   

    public class Caretaker{
        //备忘录对象
        private Memento memento;
        public Memento getMemento(){
            return memento;
        }
        public void setMemento(Memento memento){
            this.memento = memento;
        }
    }
    
现在看看是如何使用的：     

    public class Client{
        public static void main(String[] args){
            Originator originator = new Originator();
            Caretaker caretaker = new Caretaker();
            caretaker.setMemento(originator.createMemento()); 
            originator.restoreMemento(caretaker.getMemento);
        }
    }