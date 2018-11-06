---
layout: post
title: "设计模式：构建者（builder） in java"
date: 2012-11-05 20:08
comments: true
categories: 设计模式
tags: [ Java, builder ]
---
**定义：**建造者模式,又叫生成器模式，将一个复杂对象的构建与它的表示分离,使得同样的构建过程可以创建不同的表示.
**适用性**:
>a.当创建复杂对象的算法应该独立于该对象的组成部分以及他们的装配方式.
>b.当构建过程必须允许构造的对象有不同的表示.

构建者模式的**核心思想**:
>将一个“**复杂对象的构建算法**”与它的“**部件及组装方式**”分离，使得构件算法和组装方式可以独立应对变化；复用同样的构建算法可以创建不同的表示，不同的构建过程可以复用相同的部件组装方式。


建造者者模式有4个角色：

* Product产品类:
要构建的对象
* Builder抽象构建者:
定义操作接口
* ConcreteBuilder具体建造者:
实现Builder所有方法
* Director导演类：
每个观察者在接收到消息后的更新操作是不同的。

<!--more-->
下面以参考文献里提到的造人来说明:
首先定义以下产品类person,本例的产品就是"人"了.

    public class Person {


        private List<String> arms = new ArrayList<String>();

        private List<String> heads = new ArrayList<String>();

        private List<String> legs = new ArrayList<String>();

        private List<String> bodys = new ArrayList<String>();

        public Person(){
            System.out.println("person开始构建......");
        }

        @Override
        public String toString() {
            return "我有"+heads.size()+"个头,"+bodys.size()+"个身体,"+arms.size()+"个手臂,"+legs.size()+"个腿.";
        }

        //setter and getter ...
    }
人嘛,一般都有头,身体,手臂,腿了.

下面是一个抽象的Builder:

    public  abstract class Builder {

        protected Person person = new Person();

        public abstract void buildHead();
        public abstract void buildArm();
        public abstract void buildLeg();
        public abstract void buildBody();


        public abstract Person getPerson();
    }
这里面定义了如何装配头,身体,手臂,腿的方法.
到这里,我可能想构建一个普通人:

    public class HumanBuilder extends Builder {

        @Override
        public void buildHead() {
            person.getHeads().add("human head");
        }

        @Override
        public void buildArm() {
            person.getArms().add("human arm");
        }

        @Override
        public void buildLeg() {
            person.getLegs().add("human leg");
        }

        @Override
        public void buildBody() {
            person.getBodys().add("human body");
        }

        @Override
        public Person getPerson() {
            return person;
        }
    }
在这个类里,定义了实际的构建过程,需要注意的是各个部分可都是"human"的.
另外,我如果想构建另外一种人呢,比方奥特曼,那自然需要另外的一个Builder了.

    public class UltramanBuilder extends Builder {

        @Override
        public void buildHead() {
            person.getHeads().add("ultraman head");
        }

        @Override
        public void buildArm() {
            person.getArms().add("ultraman arm");
        }

        @Override
        public void buildLeg() {
            person.getLegs().add("ultraman leg");
        }

        @Override
        public void buildBody() {
            person.getBodys().add("ultraman body");
        }

        @Override
        public Person getPerson() {
            return person;
        }
    }
接下了,就是导演要出场了,到底是要一个普通人还是奥特曼呢,一切都是导演说了算.

    public class PersonDirector {

        private Builder builder;

        public PersonDirector(Builder builder){
            this.builder = builder;
        }

        public Person createPerson(){
            builder.buildBody();
            builder.buildLeg();
            builder.buildLeg();
            if (builder instanceof HumanBuilder){
                builder.buildHead();
                builder.buildArm();
            }else if(builder instanceof UltramanBuilder){
                for(int i=0;i<3;i++){//3 head
                    builder.buildHead();
                }
                for(int i=0;i<6;i++){//6 arms
                    builder.buildArm();
                }

            }
            return builder.getPerson();
        }
    }
奥特曼可不像普通人一样,那可是有3头6臂的.

到这里构建者模式已经好了,到具体场景里用一下吧.

    public class Client {
        public static void main(String[] args) {
            PersonDirector personDirector = null;
            Person person = null;
            Builder builder = null;
            builder = new HumanBuilder();
            personDirector = new PersonDirector(builder);
            person = personDirector.createPerson();
            System.out.println(person);

            builder = new UltramanBuilder();
            personDirector = new PersonDirector(builder);
            person = personDirector.createPerson();
            System.out.println(person);
        }
    }
运行结果如下:

    person开始构建......
    我有1个头,1个身体,1个手臂,2个腿.
    person开始构建......
    我有3个头,1个身体,6个手臂,2个腿.

可见,不同的builder构建出的是不同的人.完全满足需要.

**需要注意**
* 构建算法由Director来确定,Builder只负责提供装配接口.
* 组成不见不能替换,比如普通人的头给奥特曼的头明显是不一样的.

**Builder in Java**:

在Java中有一个StringBuilder,看名字应该就知道了.

       //Client同时充当了Director的角色
       StringBuilder builder = new StringBuilder();
       builder.Append("www");
       builder.Append(".shenyanchao");
       builder.Append(".cn");
       //返回string对象：www.shenyanchao.cn
       builder.toString();

这中间没有了director,用client进行了代替.StringBuilder既是抽象接口又是具体构建者. 返回的字符串自然就是产品了. 这是一种简单的Builder模式.


参考文档:
<http://www.cnblogs.com/happyhippy/archive/2010/09/01/1814287.html>
