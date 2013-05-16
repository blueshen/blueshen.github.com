---
layout: post
title: "apache mpm介绍"
date: 2012-10-12 12:45
comments: true
categories: apache
tags: [ apache, mpm, worker, perfork  ]
---
##什么是MPM？ 

MPM（Multi -Processing Modules，多路处理模块）是Apache2.x中影响性能的最核心特性。 

是Apache 2.x才支持的一个可插入的并发模型，在编译的时候，我们只可以选择一个并发模型。 

配置文件：/usr/local/apache2/conf/extra/httpd-mpm.conf 

如果apache是默认安装的可能配置在httpd.conf文件中。根据实际情况查找配置。 

使用格式： 
进入apache的目录，对apache进行如下编译： 

Linux代码 
`./configure --help|grep mpm  `
<!--more-->

显示内容如下： 

Linux代码   
	--with-mpm=MPM  
	Choose the process model for Apache to use.  
	MPM={beos|worker|prefork|mpmt_os2| perchild|leader|threadpool}  

* 1、Beos、mpmt_os2分别是BeOS和OS/2上缺省的MPM。 

* 2、perchild主要设计目的是以不同的用户和组的身份来运行不同的子进程，这在运行多个需要CGI的虚拟主机时特别有用，会比1.3版中的SuExec 机制做得更好。 

* 3、leader和threadpool都是基于worker的变体，还处于实验性阶段，某些情况下并不会按照预期设想的那样工作，所以 Apache官方也并不推荐使用。 

* 4、prefork如果不用“–with-mpm ”显式指定某种MPM，prefork就是LInux/Unix平台上缺省的MPM.它所采用的预派生子进程方式也是 Apache 1.3中采用的模式.prefork本身并没有使用到线程，2.0版使用它是为了与1.3版保持兼容性；另一方面，prefork用单独的子进程来处理不同的请求，进程之间是彼此独立的，这也使其成为最稳定的MPM之一。  
prefork的工作原理是，控制进程在最初建立“StartServers”个子进程后,为了满足MinSpareServers设置的需要创建一个进程,等待一秒钟，继续创建两个，再等待一秒钟，继续创建四个……如此按指数级增加创建的进程数,最多达到每秒32个，直到满足MinSpareServers设置的值为止。这就是预派生（prefork）的由来.这种模式可以不必在请求到来时再产生新的进程，从而减小了系统开销以增加性能。 

* 5、worker相对于prefork,worker是2.x版中全新的支持多线程和多进程混合模型的MPM。由于使用线程来处理,所以可以处理相对海量的请求，而系统资源的开销要小于基于进程的服务器.但是，worker也使用了多进程，每个进程又生成多个线程，以获得基于进程服务器的稳定性.这种MPM的工作方式将是Apache 2.x的发展趋势。    
worker的工作原理是，由主控制进程生成“StartServers”个子进程，每个子进程中包含固定的ThreadsPerChild 线程数，各个线程独立地处理请求。同样，为了不在请求到来时再生成线程，MinSpareThreads和MaxSpareThreads设置了最少和最多的空闲线程数；而MaxClients设置了所有子进程中的线程总数.如果现有子进程中的线程总数不能满足负载，控制进程将派生新的子进程。 

##如何判断当前的服务器使用那种MPM 模块? 
若使用prefork，在make编译和make install安装后，使用“httpd -l”来确定当前使用的MPM， 
如下示:    

	[aaron@webslave1 extra]$ /usr/local/apache2/bin/httpd -l 
	Compiled in modules: 
	core.c 
	...... 
	prefork.c 
	...... 
应该会看到prefork.c（如果看到worker.c说明使用的是worker MPM，依此类推）。再查看缺省生成的httpd.conf配置文件，里面包含如下配置段： 

Linux代码     

	<IfModule prefork.c>  
	StartServers 5  
	MinSpareServers 5  
	MaxSpareServers 10  最大的空闲进程数 
	MaxClients 150  Apache可以同时处理的请求(最重要)--即为常说的并发连接数!! 
	MaxRequestsPerChild 0  每个子进程可处理的请求数 
	</IfModule>  


MaxSpareServers设置了最大的空闲进程数，如果空闲进程数大于这个值，Apache会自动kill掉一些多余进程。这个值不要设得过大，但如果设的值比MinSpareServers小，Apache会自动把其调整为MinSpareServers+1。如果站点负载较大，可考虑同时加大MinSpareServers和MaxSpareServers。 

MaxRequestsPerChild设置的是每个子进程可处理的请求数。每个子进程在处理了“MaxRequestsPerChild”个请求后将自动销毁。0意味着无限，即子进程永不销毁。虽然缺省设为0可以使每个子进程处理更多的请求，但如果设成非零值也有两点重要的好处： 

可防止意外的内存泄漏； 
在服务器负载下降的时侯会自动减少子进程数。 

因此，可根据服务器的负载来调整这个值。笔者认为10000左右比较合适。 

MaxClients是这些指令中最为重要的一个，设定的是Apache可以同时处理的请求，是对Apache性能影响最大的参数。  
其缺省值150是远远不够的，如果请求总数已达到这个值（可通过ps -ef|grep http|wc -l来确认），那么后面的请求就要排队，直到某个已处理请求完毕。这就是系统资源还剩下很多而HTTP访问却很慢的主要原因。系统管理员可以根据硬件配置和负载情况来动态调整这个值。  
虽然理论上这个值越大，可以处理的请求就越多，但Apache默认的限制不能大于256。如果把这个值设为大于256，那么Apache将无法起动。事实上，256对于负载稍重的站点也是不够的。在Apache 1.3中，这是个硬限制。如果要加大这个值，必须在“configure”前手工修改的源代码树下的src/include/httpd.h中查找256，就会发现“#define HARD_SERVER_LIMIT 256”这行。把256改为要增大的值（如4000），然后重新编译Apache即可。在Apache 2.0中新加入了ServerLimit指令，使得无须重编译Apache就可以加大MaxClients。下面是笔者的prefork配置段： 

Linux代码     

	<IfModule prefork.c>  
	StartServers 10  
	MinSpareServers 10  
	MaxSpareServers 15  
	ServerLimit 2000  
	MaxClients 1000  
	MaxRequestsPerChild 10000  
	</IfModule>  


　上述配置中，ServerLimit的最大值是20000，对于大多数站点已经足够。如果一定要再加大这个数值，对位于源代码目录下 
/httpd-2.2.15/server/mpm/prefork/prefork.c中以下两行做相应修改即可：  
Linux代码    

	#define DEFAULT_SERVER_LIMIT 256  
	#define MAX_SERVER_LIMIT 20000  

worker的工作原理是，由主控制进程生成“StartServers”个子进程，每个子进程中包含固定的ThreadsPerChild线程数，各个线程独立地处理请求。同样，为了不在请求到来时再生成线程，MinSpareThreads和MaxSpareThreads设置了最少和最多的空闲线程数；而MaxClients设置了所有子进程中的线程总数。如果现有子进程中的线程总数不能满足负载，控制进程将派生新的子进程。 

MinSpareThreads和MaxSpareThreads的最大缺省值分别是75和250。这两个参数对Apache的性能影响并不大，可以按照实际情况相应调节。 
ThreadsPerChild是worker MPM中与性能相关最密切的指令。ThreadsPerChild的最大缺省值是64，如果负载较大，64也是不够的。这时要显式使用ThreadLimit指令，它的最大缺省值是20000。上述两个值位于源码树server/mpm/worker/worker.c中的以下两行： 

Linux代码    

	#define DEFAULT_THREAD_LIMIT 64  
	#define MAX_THREAD_LIMIT 20000  

这两行对应着ThreadsPerChild和ThreadLimit的限制数。最好在configure之前就把64改成所希望的值。注意，不要把这两个值设得太高，超过系统的处理能力，从而因Apache不起动使系统很不稳定。 
Worker模式下所能同时处理的请求总数是由子进程总数乘以ThreadsPerChild值决定的，应该大于等于MaxClients。如果负载很大，现有的子进程数不能满足时，控制进程会派生新的子进程。默认最大的子进程总数是16，加大时也需要显式声明ServerLimit（最大值是20000）。这两个值位于源码树server/mpm/worker/worker.c中的以下两行： 

Linux代码     

	#define DEFAULT_SERVER_LIMIT 16  
	#define MAX_SERVER_LIMIT 20000  

需要注意的是，如果显式声明了ServerLimit，那么它乘以ThreadsPerChild的值必须大于等于MaxClients，而且MaxClients必须是ThreadsPerChild的整数倍，否则Apache将会自动调节到一个相应值（可能是个非期望值）。下面是笔者的worker配置段： 

Linux代码     

	<IfModule worker.c>  
	StartServers 3  
	MaxClients 2000  
	ServerLimit 25  
	MinSpareThreads 50  
	MaxSpareThreads 200  
	ThreadLimit 200  
	ThreadsPerChild 100  
	MaxRequestsPerChild 0  
	</IfModule>  

硬性公式：  

         ThreadLimit >= ThreadsPerChild
         MaxClients <= ServerLimit * ThreadsPerChild 必须是ThreadsPerChild的倍数
         MaxSpareThreads >= MinSpareThreads+ThreadsPerChild
ServerLimit默认是16,最大20000;ThreadLimit默认是64,最大20000。  
通过上面的叙述，可以了解到Apache 2.0中prefork和worker这两个重要MPM的工作原理，并可根据实际情况来配置Apache相关的核心参数，以获得最大的性能和稳定性。