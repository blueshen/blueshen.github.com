---
layout: post
title: "MySql性能监控"
date: 2013-01-10 19:03
comments: true
categories: mysql
tags: [ mysql, monitor, slowquery ]
---

##用mysqldumpslow分析mysql的slow query log   
mysql有一个功能就是可以log下来运行的比较慢的sql语句，默认是没有这个log的，为了开启这个功能，要修改my.cnf或者在mysql启动的时候加入一些参数。如果在my.cnf里面修改，需增加如下几行   

	long_query_time = 1
	log-slow-queries = /var/youpath/slow.log
	log-queries-not-using-indexes[这个在mysql4.10以后才被引入]

long_query_time 是指执行超过多久的sql会被log下来，这里是1秒。  
log-slow-queries 设置把日志写在那里，可以为空，系统会给一个缺省的文件host_name-slow.log，我生成的log就在mysql的data目录   
log-queries-not-using-indexes 就是字面意思，log下来没有使用索引的query。   
把上述参数打开，运行一段时间，就可以关掉了，省得影响生产环境。  
<!--more-->
接下来就是分析了，我这里的文件名字叫host-slow.log。     
先mysqldumpslow –help以下，我主要用的是   

	-s ORDER what to sort by (t, at, l, al, r, ar etc), ‘at’ is default
	-t NUM just show the top n queries
	-g PATTERN grep: only consider stmts that include this string
-s，是order的顺序，说明写的不够详细，俺用下来，包括看了代码，主要有
c,t,l,r和ac,at,al,ar，分别是按照query次数，时间，lock的时间和返回的记录数来排序，前面加了a的时倒叙    
-t，是top n的意思，即为返回前面多少条的数据    
-g，后边可以写一个正则匹配模式，大小写不敏感的   

	mysqldumpslow -s c -t 20 host-slow.log
	mysqldumpslow -s r -t 20 host-slow.log
上述命令可以看出访问次数最多的20个sql语句和返回记录集最多的20个sql。  
  
	mysqldumpslow -t 10 -s t -g “left join” host-slow.log    
这个是按照时间返回前10条里面含有左连接的sql语句。   
用了这个工具就可以查询出来那些sql语句是性能的瓶颈，进行优化，比如加索引，该应用的实现方式等。      
 
##linux 系统整体性能查看的方法:
vmstat 10 -----每10秒刷新一次
 
	procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu------   
 	r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st   
	0  0      0  56508  10968  68380    0    0   184    40 1021   88  3 14 78  5  0    
	0  0      0  56508  10976  68376    0    0     0     3 1251   49  0  0 100  0  0    
 	0  0      0  56508  10976  68376    0    0     0     0 1250   50  0  0 100  0  0    
 	0  0      0  56508  10984  68376    0    0     0     4 1251   51  0  0 100  0  0    
 	0  0      0  56508  10984  68376    0    0     0     0 1250   48  0  0 100  0  0    
 	0  0      0  56508  10984  68376    0    0     0     0 1250   50  0  0 100  0  0   
 	0  0      0  56508  10984  68376    0    0     0     0 1250   51  0  0 100  0  0      
 	0  0      0  56508  10992  68376    0    0     0     2 1250   49  0  0 100  0  0    
 	0  0      0  56508  10992  68376    0    0     0     0 1250   51  0  0 100  0  0   
 
procs:   
r-->;在运行队列中等待的进程数    
b-->;在等待io的进程数   
w-->;可以进入运行队列但被替换的进程    

memory    
swap-->;现时可用的交换内存（k表示）   
free-->;空闲的内存（k表示）   

pages   
re－－》回收的页面   
mf－－》非严重错误的页面   
pi－－》进入页面数（k表示）   
po－－》出页面数（k表示）   
fr－－》空余的页面数（k表示）   
de－－》提前读入的页面中的未命中数   
sr－－》通过时钟算法扫描的页面    

disk 显示每秒的磁盘操作。 s表示scsi盘，0表示盘号    

fault 显示每秒的中断数   
in－－》设备中断    
sy－－》系统中断   
cy－－》cpu交换   

cpu 表示cpu的使用状态   
cs－－》用户进程使用的时间   
sy－－》系统进程使用的时间   
id－－》cpu空闲的时间   


其中:   
如果 r经常大于 4 ，且id经常少于40，表示cpu的负荷很重。   
如果pi，po 长期不等于0，表示内存不足。   
如果disk 经常不等于0， 且在 b中的队列 大于3， 表示 io性能不好。   
 
每100s显示一次mysql 运行的状态:   

	mysqladmin extended -i 100 –r
 
显示mysql服务器的线程列表    

	mysqladmin -u root -p process    
	Enter password:
	+----+------+-----------+----+---------+------+-------+------------------+
	| Id | User | Host      | db | Command | Time | State | Info             |
	+----+------+-----------+----+---------+------+-------+------------------+
	| 12 | root | localhost |    | Query   | 0    |       | show processlist |
	+----+------+-----------+----+---------+------+-------+------------------+
 
相关命令：
 
一，获取mysql用户下的进程总数    
	ps -ef | awk '{print $1}' | grep "mysql" | grep -v "grep" | wc-1
二，主机性能状态    
	[root@ ~]# uptime
 13:05:52 up 53 days, 52 min,  1 user,  load average: 0.00, 0.00, 0.00
 
三，CPU使用率   
top 或 vmstat    
四，磁盘IO量    
vmstat 或  iostat   
五，swap进出量[内存]   
free    
六，数据库性能状态   
(1)QPS(每秒Query量)   
QPS = Questions(or Queries) / seconds    
mysql > show status like 'Question';   
(2)TPS(每秒事务量)   
TPS = (Com_commit + Com_rollback) / seconds
mysql > show status like 'Com_commit';    
mysql > show status like 'Com_rollback';   
(3)key Buffer 命中率    
key_buffer_read_hits = (1-key_reads / key_read_requests) * 100%   
key_buffer_write_hits = (1-key_writes / key_write_requests) * 100%    
mysql> show status like 'Key%';   
(4)InnoDB Buffer命中率    
innodb_buffer_read_hits = (1 - innodb_buffer_pool_reads / innodb_buffer_pool_read_requests) * 100%    
mysql> show status like 'innodb_buffer_pool_read%';     
(5)Query Cache命中率    
Query_cache_hits = (Qcahce_hits / (Qcache_hits + Qcache_inserts )) * 100%;    
mysql> show status like 'Qcache%';   
(6)Table Cache状态量    
mysql> show status like 'open%';    
(7)Thread Cache 命中率    
Thread_cache_hits = (1 - Threads_created / connections ) * 100%    
mysql> show status like 'Thread%';    
mysql> show status like 'Connections';     
(8)锁定状态     
mysql> show status like '%lock%';    
(9)复制延时量    
mysql > show slave status    
(10) Tmp Table 状况(临时表状况)   
mysql > show status like 'Create_tmp%';    
(11) Binlog Cache 使用状况    
mysql > show status like 'Binlog_cache%';   
(12) Innodb_log_waits 量    
mysql > show status like 'innodb_log_waits';     