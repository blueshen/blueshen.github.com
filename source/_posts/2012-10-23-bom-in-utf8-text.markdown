---
layout: post
title: "什么是BOM(Byte Order Mark)?"
date: 2012-10-23 17:05
comments: true
categories: encode
tags: [ UTF8, encode, BOM, XML ]
---
###引子###
在使用诸如UltraEditor,NotePad++等编辑工具时，经常会遇到encode进行转换的情况。以NotePad++为例，一篇ASCII编码的文本，通过菜单栏Encoding->convert to **进行编码格式的转换。我却发现一个奇怪的问题，下拉框里有这样两个选项：
   
* Convert to UTF-8   
* Convert to UTF-8 without BOM   

同样的是转化为UTF-8编码，为什么还牵涉到BOM呢。
<!--more-->

###原因###

1、big endian和little endian  
 
big endian和little endian是CPU处理多字节数的不同方式。例如“汉”字的Unicode编码是6C49。那么写到文件里时，究竟是将6C写在前面，还是将49写在前面？如果将6C写在前面，就是big endian。还是将49写在前面，就是little endian。那么在读文件的时间 “endian”这个词出自《格列佛游记》。小人国的内战就源于吃鸡蛋时是究竟从大头(Big-Endian)敲开还是从小头(Little-Endian)敲开，由此曾发生过六次叛乱，其中一个皇帝送了命，另一个丢了王位。 
我们一般将endian翻译成“字节序”，将big endian和little endian称作“大尾”和“小尾”。  
UTF-8以字节为编码单元，没有字节序的问题。UTF-16以两个字节为编码单元，在解释一个UTF-16文本前，首先要弄清楚每个编码单元的字节序。例如收到一个“奎”的Unicode编码是594E，“乙”的Unicode编码是4E59。如果我们收到UTF-16字节流“594E”，那么这是“奎”还是“乙”？  
 
2.如何解决以上问题？   

在UCS 编码中有一个叫做"ZERO WIDTH NO-BREAK SPACE"的字符，它的编码是FEFF。而FFFE在UCS中是不存在的字符，所以不应该出现在实际传输中。UCS规范建议我们在传输字节流前，先传输字符"ZERO WIDTH NO-BREAK SPACE"。这样如果接收者收到FEFF，就表明这个字节流是Big-Endian的；如果收到FFFE，就表明这个字节流是Little-Endian的。因此字符"ZERO WIDTH NO-BREAK SPACE"又被称作BOM。  

UTF-8不需要BOM来表明字节顺序，但可以用BOM来表明编码方式。字符"ZERO WIDTH NO-BREAK SPACE"的UTF-8编码是EF BB BF。所以如果接收者收到以EF BB BF开头的字节流，就知道这是UTF-8编码了。

3.BOM在XML中的使用   

 W3C定义了三条XML解析器如何正确读取XML文件的编码的规则：   

* 如果文档有BOM(字节顺序标记，一般来说，如果保存为unicode格式，则包含BOM，ANSI则无)，就定义了文件编码   
* 如果没有BOM，就查看XML声明的编码属性   
* 如果上述两个都没有，就假定XML文挡采用UTF-8编码   