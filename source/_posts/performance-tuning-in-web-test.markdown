---
layout: post
title: "Web系统性能调优常用技巧"
date: 2013-09-24 18:02
comments: true
categories: 调优
tags: [ apache, tomcat, mysql, linux, 调优 ]
---
#### 目标Web系统
apache + tomcat + mysql + linux, 介绍如何定位瓶颈与调优

#### Tomcat
按照官方默认配置。
并发150正常;达到210时，报connection refuse

$TOMCAT_HOME/conf/server.xml中

```xml
<Connector port="8080" protocol="HTTP/1.1"      connectionTimeout="20000"
               redirectPort="8443" URIEncoding="UTF-8"/>
```

maxThreads 默认值是200
acceptCount 默认值是100

解决方法：显式指明以上2项，并提高数值。并可配合min/maxSpareThreads

参数文档： <http://tomcat.apache.org/tomcat-6.0-doc/config/http.html>

<!--more-->
继续增加并发数，出现以下现象：

- 420并发，观察到Jmeter的聚合报告有明显卡顿现象。
- Noah监控，显示CPU使用率下降。大锯齿出现。

  问题分析：
  ​      在加压过程中，对tomcat的JVM情况进行监控。出现FullGC,每次大概4s .

如何监控GC吗？

	jstat -gcutil 3950 3000 5
	
	  S0     S1         E          O      P       YGC     YGCT    FGC       FGCT     GCT
	  0.00  47.86  66.10   6.55  99.92      7       0.087     0         0.000    0.087
	  0.00  47.86  66.90   6.55  99.94      7       0.087     0         0.000    0.087
	  0.00  47.86  67.30   6.55  99.94      7       0.087     0         0.000    0.087
	  0.00  47.86  67.30   6.55  99.94      7       0.087     0         0.000    0.087
	  0.00  47.86  67.30   6.55  99.94      7       0.087     0         0.000    0.087

解决方法：
​         $TOMCAT_HOME/bin/catalina.sh   第一行添加
​    	export CATALINA_OPTS="-Xmx3500m -Xms2048m -XX:PermSize=256m XX:MaxPermSize=512m -Xss128k"

JAVA_OPTS   VS   CATALINA_OPTS（推荐使用）
**差别**：JAVA_OPTS start run stop   适用所有JVM
​               CATALINA_OPTS   start run    专门配置tomcat

tomcat默认-client ，production环境加-server

JVM参数文档：<http://kenwublog.com/docs/java6-jvm-options-chinese-edition.htm>

##### Tomcat executor
据信使用executor后，能在实际中有更好的性能以及稳定性！更为重要的是能在多个connector之间共用。

```xml
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-" maxThreads="1000" minSpareThreads="25"/>

<Connector executor="tomcatThreadPool"  port="8080" protocol="HTTP/1.1"   connectionTimeout="20000"  redirectPort="8443" />
```

此时，connector再使用maxThreads等属性将被忽略。

##### Tomcat Connector protocol

- bio   默认模式，性能低下，无任何优化，阻塞IO模式

    protocol = "org.apache.coyote.http11.Http11Protocol" or HTTP/1.1(for http connector)
    protocol = "org.apache.jk.server.JkCoyoteHandler" or AJP/1.3(for ajp connector)

- nio   Java的异步io技术，非阻塞 IO

    protocol = "org.apache.coyote.http11.Http11NioProtocol"(for http connector)
    protocol = "org.apache.coyote.ajp.AjpProtocol"(for ajp connector)

- apr  需要安装apr(Apache Portable Runtime)和Native。

    protocol = "org.apache.coyote.http11.Http11AprProtocol"(for http connector)
    protocol = "org.apache.coyote.ajp.AjpAprProtocol"  (for ajp connector)

参考文档：
<http://tomcat.apache.org/tomcat-6.0-doc/config/http.html#Connector Comparison>

<http://tomcat.apache.org/tomcat-6.0-doc/config/ajp.html>

##### MySQL

大并发下，响应时间如何提高，绝大部分瓶颈都处于数据库端。

与数据库的连接，使用JDBC连接池

slow query监控：
1. 开启慢查询 my.cnf

    long_query_time = 2
    log-slow-queries =  slow.log

2.使用mysqldumpslow分析log

```shell
mysqldumpslow -t 10 -s t  slow.log
```

解决：

- 1.SQL调优   （http://coolshell.cn/articles/1846.html）
- 2.走类似lucene索引，空间换时间（idea实际测试中，吞吐提升7倍）
- 3.使用NoSQL?

$MYSQL_HOME/etc/my.cnf  或者 my.ini（windows）

主要影响性能参数：

    max-connections = 3000     会话数上限
    max_connect_errors = 10000    允许的最大连接错误量
    query_cache_size =128M    查询缓存
    query_cache_limit = 2M   小于这么大的才缓存，保护查询缓存
    sort_buffer_size =256M    排序order by 或 group by 使用

参考：<http://www.ha97.com/4110.html>

#### Apache

处理静态资源，无法处理动态（需要应用服务器支持）

静态资源直接交给apache处理

ab 命令进行测试，达到1000并发很easy

	ab -k -c 1000  -n 1000000  http://hostname:port/path

参考：<http://httpd.apache.org/docs/2.2/programs/ab.html>

##### 开启MPM支持大并发

<table border="1px">
<thead>
<th>Mpm模式</th><th>并发方式</th><th>内存占用</th><th>并发性能</th>
</thead>
<tbody>
<tr>
<td>prefork</td><td>进程</td><td>高</td><td>低并发下，吞吐率高</td>
</tr>
<tr>
<td>worker</td><td>进程+线程</td><td>低</td><td>支持海量并发</td>
</tr>
</tbody>
</table>

确定apache模式命令：

	./httpd –l
输出：

    Compiled in modules:
    core.c
    worker.c
    http_core.c
    mod_so.c

修改httpd-mpm.conf

```xml
<IfModule mpm_worker_module>
    StartServers          2
    MaxClients          150
    MinSpareThreads      25
    MaxSpareThreads      75
    ThreadsPerChild      25
    MaxRequestsPerChild   0
</IfModule>
```

上面的配置需要满足以下公式：
​         ThreadLimit >= ThreadsPerChild
​         MaxClients <= ServerLimit * ThreadsPerChild 必须是ThreadsPerChild的倍数
​         MaxSpareThreads >= MinSpareThreads+ThreadsPerChild

#### Linux
##### too many open files error
`ulimit -a `进行查看
修改`vi /etc/security/limits.conf`

添加：

    *    -     nofile    65535

参考文档：
 <http://blog.csdn.net/lifeibo/article/details/5972356>

Http连接是基于TCP的，这个时候需要对linux服务器进行优化。
**三次握手**

![三次握手](/images/blog/2013/three-times-handshake.png)

**四次挥手**

![四次挥手](/images/blog/2013/four-wave.png)

如何查看服务器TCP状态？
命令：

    netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
输出：

    ESTABLISHED 1423
    FIN_WAIT1 1
    FIN_WAIT2 262
    SYN_SENT 1
    TIME_WAIT 962

优化TCP连接相关参数：

`vi /etc/sysctl.conf`

    net.ipv4.tcp_fin_timeout = 30  保持在FIN-WAIT-2的时间。默认60秒。2.2内核是180秒
    net.ipv4.tcp_keepalive_time = 1200     长连接keepalive打开，发送的频率。默认7200（2H）
    net.ipv4.tcp_tw_reuse = 1     默认0，处于TIME-WAIT状态的socket可以用于新的TCP连接
    net.ipv4.tcp_tw_recycle = 1   默认0，TIME-WAIT状态的sockets快速回收
    net.ipv4.ip_local_port_range = 1024    65000 向外连接的端口范围，默认32768~61000
    net.ipv4.tcp_max_syn_backlog = 8192  默认1024/128,未获得客户端连接请求并保存在队列中的最大数目。
    net.ipv4.tcp_max_tw_buckets = 5000   默认180000，系统处理的最大TIME_WAIT数目。
    net.ipv4.route.gc_timeout = 100  路由缓存刷新频率，失败多长时间跳到另一个。默认300.
    net.ipv4.tcp_syncookies = 1	默认0，SYN队列溢出，启用cookies处理。
    net.ipv4.tcp_syn_retries = 1       新建连接发送SYN次数，默认5，180秒
    net.ipv4.tcp_synack_retries = 1    3次握手的第二步，重试几次。默认5.


`/sbin/sysctl -p`   生效

参考文档：<http://www.itlearner.com/article/4792>

备注：此文由本人在公司内做的PPT分享制作而成，内容有些省略以及跳跃。欢迎留言。
