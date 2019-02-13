---
layout: post
title: "设计模式：解释器（interpreter） in java"
date: 2012-11-02 20:09
comments: true
categories: 设计模式
tags: [ interpreter, pattern, 解释器, 设计模式 ]
---
**定义：**解释器模式是一种按照规定语法进行解析的方案，在现在项目中使用较少。正式定义：给定一门语言，定义它的文法的一种表示，并定义一个解释器，该解释器使用该表示来解释语句中的句子。    

解释器模式有以下几个角色：    

- AbstractExpression抽象解释器     
具体的解释任务由各个实现类完成。
- TerminalExpression终结符表达式    
实现与文法中的元素相关联的解释操作，通常一个解释器模式中只有一个终结符表达式，但有多个实例，对应不同的终结符。
- NonterminalExpression非终结符表达式     
文法中的每条规则对应于一个非终结表达式。非终结符表达式根据逻辑的复杂程度而增加，原则上每个文法规则都对应一个非终结符表达式。   
- Context环境角色

