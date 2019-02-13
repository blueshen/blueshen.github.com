---
layout: post
title: "设计模式：状态（state） in java"
date: 2012-11-02 20:11
comments: true
categories: 设计模式
tags: [ state, pattern, 状态, 设计模式 ]
---
**定义：**当一个对象内在状态改变时允许其改变行为，这个对象看起来想改变了其类。    
状态模式的核心是封装，状态的变更引起了行为的变更，从外部看起来就好像这个对象对应的类发生了改变一样。    

状态模式有3个角色：    

- State抽象状态角色      
接口或抽象类，负责对象状态定义，并且封装环境角色以实现状态转换。    
- ConcreteState具体状态角色    
每一个具体状态必须完成2个职责：本状态的行为管理以及趋向状态处理。简单说，就是本状态下要做的事情，以及本状态如何过渡到其他状态。       
- Context环境角色     
定义客户端需要的接口，并且负责具体状态的切换。    

<!--more-->
具体看看各个角色的实现.    

抽象状态角色    

    public abstract class State{
        //定义一个环境角色，提供子类访问
        protected Context context;
        //设置环境角色
        public void setContext(Context _context){
            this.context = _context;
        }
        //行为1
        public abstract void handle1();
        //行为2
        public abstract void handle2();
    
    }

具体状态角色    

    public class ConcreteState1 extends State{
        @override
        public void handle1(){
            //本状态下必须处理的逻辑
        }
        @override
        public void handle2(){
            //设置当前状态为STATE2
            super.context.setCurrentState(Context.STATE2);
            //过渡到STATE2状态，由Context实现
            super.context.handle2();
        }
    }
    
    public class ConcreteState2 extends State{
    
        @override
        public void handle1(){
            //设置当前状态为STATE1
            super.context.setCurrentState(Context.STATE1);
            //过渡到STATE1状态，由Context实现
            super.context.handle1();
        }
        @override
        public void handle2(){
            //本状态下必须处理的逻辑
        }
    }

具体环境角色     

    public class Context{
        //定义状态
        public final static State STATE1 = new ConcreteState1();
        public final static State STATE2 = new ConcreteState2();
        //当前状态
        private State CurrentState;
        //获得当前状态
        public State getCurrentState(){
            return CurrentState;
        }
        //设置当前状态
        public void setCurrentState(State currentState){
            this.CurrentState = currentState;
            //切换状态
            this.CurrentState.setContext(this);
        }
        //行为委托
        public void handle1(){
            this.CurrentState.handle1();
        }
        public void handle2(){
            this.CurrentState.handle2();
        }
    }
环境角色有2个不成文的约束：    

- 把状态对象声明为静态常量，有几个状态对象就声明几个静态常量。 
- 环境角色具有状态抽象角色定义的所有行为，具体执行使用委托方式。   

在具体使用状态模式的时候，直接调用Context就行了。    

    public class Client{
        public static void main(String[] args){
            Context context = new Context();
            context.setCurrentState(new ConcreteState1());
            context.handle1();
            context.handle2();
        }
    }






