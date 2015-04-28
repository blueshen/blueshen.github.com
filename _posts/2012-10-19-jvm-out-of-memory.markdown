---
layout: post
title: "Java内存溢出详解"
date: 2012-10-19 23:12
comments: true
categories: JVM
tags: [ JVM, Tomcat, GC ]
---

 
一、常见的Java内存溢出有以下三种：
 
1. `java.lang.OutOfMemoryError: Java heap space` ----JVM Heap（堆）溢出
JVM在启动的时候会自动设置JVM Heap的值，其初始空间(即-Xms)是物理内存的1/64，最大空间(-Xmx)不可超过物理内存。
可以利用JVM提供的-Xmn -Xms -Xmx等选项可进行设置。Heap的大小是Young Generation 和Tenured Generaion 之和。
在JVM中如果98％的时间是用于GC，且可用的Heap size 不足2％的时候将抛出此异常信息。
解决方法：手动设置JVM Heap（堆）的大小。    
 <!--more-->   
2. `java.lang.OutOfMemoryError: PermGen space`  ---- PermGen space溢出。 
PermGen space的全称是Permanent Generation space，是指内存的永久保存区域。
为什么会内存溢出，这是由于这块内存主要是被JVM存放Class和Meta信 息的，Class在被Load的时候被放入PermGen space区域，它和存放Instance的Heap区域不同,sun的 GC不会在主程序运行期对PermGen space进行清理，所以如果你的APP会载入很多CLASS的话，就很可能出现PermGen space溢出。
解决方法： 手动设置MaxPermSize大小
 
3. `java.lang.StackOverflowError `  ---- 栈溢出
栈溢出了，JVM依然是采用栈式的虚拟机，这个和C和Pascal都是一样的。函数的调用过程都体现在堆栈和退栈上了。
调用构造函数的 “层”太多了，以致于把栈区溢出了。
通常来讲，一般栈区远远小于堆区的，因为函数调用过程往往不会多于上千层，而即便每个函数调用需要 1K的空间(这个大约相当于在一个C函数内声明了256个int类型的变量)，那么栈区也不过是需要1MB的空间。通常栈的大小是1－2MB的。
通常递归也不要递归的层次过多，很容易溢出。
解决方法：修改程序。
 
 
二、解决方法
 
在生产环境中tomcat内存设置不好很容易出现jvm内存溢出。
 
1、 linux下的tomcat：  
修改TOMCAT_HOME/bin/catalina.sh 
在“echo "Using CATALINA_BASE: $CATALINA_BASE"”上面加入以下行： 
`JAVA_OPTS="-server -Xms256m -Xmx512m -XX:PermSize=64M -XX:MaxPermSize=128m`" 
 
2、 如果tomcat 5 注册成了windows服务，以services方式启动的，则需要修改注册表中的相应键值。
修改注册表HKEY_LOCAL_MACHINE\SOFTWARE\Apache Software Foundation\Tomcat Service Manager\Tomcat5\Parameters\Java，右侧的Options
原值为

	-Dcatalina.home="C:\ApacheGroup\Tomcat 5.0"
	-Djava.endorsed.dirs="C:\ApacheGroup\Tomcat 5.0\common\endorsed"
-Xrs
加入 -Xms256m -Xmx512m 
重起tomcat服务,设置生效
 
3、 如果tomcat 6 注册成了windows服务，或者windows2003下用tomcat的安装版，
在/bin/tomcat6w.exe里修改就可以了 。
 
 
 
4、 如果要在myeclipse中启动tomcat，上述的修改就不起作用了，可如下设置：
Myeclipse->preferences->myeclipse->servers->tomcat->tomcat×.×->JDK面板中的
Optional Java VM arguments中添加：`-Xms256m -Xmx512m -XX:PermSize=64M -XX:MaxPermSize=128m`
 
 
 
三、jvm参数说明：
 
-server:一定要作为第一个参数，在多个CPU时性能佳    
-Xms：java Heap初始大小。 默认是物理内存的1/64。   
-Xmx：java heap最大值。建议均设为物理内存的一半。不可超过物理内存。  
-XX:PermSize:设定内存的永久保存区初始大小，缺省值为64M。（我用visualvm.exe查看的）   
-XX:MaxPermSize:设定内存的永久保存区最大 大小，缺省值为64M。（我用visualvm.exe查看的）   
-XX:SurvivorRatio=2  :生还者池的大小,默认是2，如果垃圾回收变成了瓶颈，您可以尝试定制生成池设置    
-XX:NewSize: 新生成的池的初始大小。 缺省值为2M。  
-XX:MaxNewSize: 新生成的池的最大大小。   缺省值为32M。   
如果 JVM 的堆大小大于 1GB，则应该使用值：-XX:newSize=640m -XX:MaxNewSize=640m -XX:SurvivorRatio=16，或者将堆的总大小的 50% 到 60% 分配给新生成的池。调大新对象区，减少Full GC次数。  
+XX:AggressiveHeap 会使得 Xms没有意义。这个参数让jvm忽略Xmx参数,疯狂地吃完一个G物理内存,再吃尽一个G的swap。   
-Xss：每个线程的Stack大小，“-Xss 15120” 这使得JBoss每增加一个线程（thread)就会立即消耗15M内存，而最佳值应该是128K,默认值好像是512k.    
-verbose:gc 现实垃圾收集信息    
-Xloggc:gc.log 指定垃圾收集日志文件  
-Xmn：young generation的heap大小，一般设置为Xmx的3、4分之一    
-XX:+UseParNewGC ：缩短minor收集的时间    
-XX:+UseConcMarkSweepGC ：缩短major收集的时间 此选项在Heap Size 比较大而且Major收集时间较长的情况下使用更合适。   
-XX:userParNewGC 可用来设置并行收集【多CPU】   
-XX:ParallelGCThreads 可用来增加并行度【多CPU】   
-XX:UseParallelGC 设置后可以使用并行清除收集器【多CPU】   