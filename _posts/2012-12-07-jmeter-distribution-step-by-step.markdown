---
layout: post
title: "jmeter 分布式 step by step"
date: 2012-12-07 13:25
comments: true
categories: jmeter
tags: [ jmeter, 分布式, 压力测试 ]
---
###环境准备###

* Jmeter(本文以2.8为例)   
* jdk 1.5+   
* 多台机器  
(假设3台，IP分别为：10.81.14.170，10.81.14.180，10.81.14.190)    

###工作原理###
1台机器做为总控机，其他机器作为节点机。总控机器，负责将JMX脚本分发到节点机上，各个节点同时独立运行，向服务发出压力，总控机可以获取并汇总报告。   
定义：   
>总控机为**client**，我们（用户）只与这太机器打交道。或者称之为**Master**;   
节点机器为**server**,它负责真正的向服务发出压力。或者称之为**Slave**;    

<!--more-->
这个只是角度不一样，就是一个总分结构。  
###Slave配置###
假设这3台机器都作为slave，那么分别在各个机器上， 
进入%JMETER HOME%/bin/目录     
运行:   
jmeter-server.bat（windows）   
jmeter-server.sh (linux)  

###Master配置###
这里，我们以10.81.14.170作为Master,
进入%JMETER HOME%/bin/ 目录   
找到jmeter.properties文件，打开并找到remote_hosts=127.0.0.1这一行，修改为remote_hosts=127.0.0.1,10.81.14.180,10.81.14.190    
其中，IP部分指向slave,并以逗号分割。由于170这台机器同时也是slave而存在，因此直接写为127.0.0.1了。    
在目录下执行：   
jmeter.bat (windows)   
jmeter.sh (linux)   
用来打开GUI界面。  
点开运行->远程启动，将会看到这样的界面：   
![Jmeter远程启动](/images/blog/jmeter-remote.png)   
从这里就可以指定哪台slave来发压力了。当然也可以选择远程全部启动了。

###为什么要分布式发压力？###
1.单机运行受限，网络、CPU、内存读可能是瓶颈所在；   
2.Jmeter是纯Java的程序，受JVM的一些限制；    
一般情况下，依据机器配置，单机的发压量为300～600，如果需要更大的并发，就需要使用分布式的多台机器同时加压。  

###配置注意事项###
1.尽量保证各台机器之间的jmeter版本一致   
2.JDK／JRE要正确安装   
3.启动端口有可能被占用了，这个需要在启动时间指定SERVER_PORT      

---
参考文献：   
<http://jmeter.apache.org/usermanual/remote-test.html>   
<http://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.pdf>


