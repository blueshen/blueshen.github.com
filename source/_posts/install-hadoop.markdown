---
layout: post
title: "Hadoop1.2.1安装部署"
date: 2014-11-13 13:24
comments: true
categories: hadoop
tags: [ hadoop, linux]
---
### 安装需求
- Java 1.6
- ssh,sshd正常安装

确保可以ssh到localhost，并且不需要密码

    ssh localhost
 如果报错，connect to host localhost port 22:Connection refused。说明ssh-server未安装或者未启动。
 运行：

    ps -ef | grep sshd
查看sshd进程是否存在，如果不存在，说明没有安装。那么进行安装。

    sudo apt-get install openssh-server

然后再执行`ssh localhost`,如果不能无密码登陆，需要做一下操作：

        ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
        cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

<!--more-->
### 相关软件
Ubuntu Linux为例：

     sudo apt-get install ssh
     sudo apt-get install rsync

### Hadoop下载
从[Hadoop官网](http://hadoop.apache.org/releases.html)下载一个稳定版，这里就是1.2.1版本啦。

    wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tar.gz

 ### 启动Hadoop

 1.解压`tar -xzvf hadoop-1.2.1.tar.gz `，进入conf/hadoop-env.sh，设置JAVA_HOME为你的JDK目录。

    export JAVA_HOME=/path/to/java/home

 2.进入/hadoop-1.2.1目录，运行`bin\hadoop`,会显示hadoop的使用说明信息。

      Usage: hadoop [--config confdir] COMMAND
      where COMMAND is one of:
      namenode -format     format the DFS filesystem
      secondarynamenode    run the DFS secondary namenode
      namenode             run the DFS namenode
      datanode             run a DFS datanode
      dfsadmin             run a DFS admin client
      mradmin              run a Map-Reduce admin client
      fsck                 run a DFS filesystem checking utility
      fs                   run a generic filesystem user client
      balancer             run a cluster balancing utility
      oiv                  apply the offline fsimage viewer to an fsimage
      fetchdt              fetch a delegation token from the NameNode
      jobtracker           run the MapReduce job Tracker node
      pipes                run a Pipes job
      tasktracker          run a MapReduce task Tracker node
      historyserver        run job history servers as a standalone daemon
      job                  manipulate MapReduce jobs
      queue                get information regarding JobQueues
      version              print the version
      jar <jar>            run a jar file
      distcp <srcurl> <desturl> copy file or directories recursively
      distcp2 <srcurl> <desturl> DistCp version 2
      archive -archiveName NAME -p <parent path> <src>* <dest> create a hadoop archive
      classpath            prints the class path needed to get the
                           Hadoop jar and the required libraries
      daemonlog            get/set the log level for each daemon
     or
      CLASSNAME            run the class named CLASSNAME
    Most commands print help when invoked w/o parameters.
  你可以下3中模式来启动hadoop：

  - 本地(standalone)模式
  - 伪分布(Pseudo-Distributed)式
  - 全分布(Full-Distributed)式

#### 本地(Standalone)模式安装
  默认情况下，Hadoop就是单机本地模式。方便调试。
  下面是一个样例，复制conf目录到input作为输入，找出符合正则的文件，输出到output目录

        $ mkdir input
        $ cp conf/*.xml input
        $ bin/hadoop jar hadoop-examples-*.jar grep input output 'dfs[a-z.]+'
        $ cat output/*

#### 伪分布(Pseudo-Distributed)式
配置conf/core-site.xml

    <configuration>
         <property>
             <name>fs.default.name</name>
             <value>hdfs://localhost:9000</value>
         </property>
    </configuration>
配置conf/hdfs-site.xml

    <configuration>
         <property>
             <name>dfs.replication</name>
             <value>1</value>
         </property>
    </configuration>

配置conf/mapred-site.xml:

    <configuration>
         <property>
             <name>mapred.job.tracker</name>
             <value>localhost:9001</value>
         </property>
    </configuration>

  格式化一个新的distributed-filesystem

     bin/hadoop namenode -format

 启动hadoop：

    bin/start-all.sh

  此时可以通过WEB UI控制台来监控NameNode，JobTracker，TaskTracker

- NameNode - <http://localhost:50070/dfshealth.jsp/>
- JobTracker - <http://localhost:50030/jobtracker.jsp/>
- TaskTracker - <http://localhost:50060/tasktracker.jsp>

 下面使用distributed filesystem来跑样例:
 拷贝conf目录到hadoop的input目录，此时可以在控制台看到创建了目录。

    $ bin/hadoop fs -put conf input

运行:
    $ bin/hadoop jar hadoop-examples-*.jar grep input output 'dfs[a-z.]+'

检查结果：

拷贝output目录到本地并检查:
    $ bin/hadoop fs -get output output
    $ cat output/*

或者直接在distributed filesystem查看:

    $ bin/hadoop fs -cat output/*

使用完，可以这样来关闭：

    $ bin/stop-all.sh


#### Hadoop分布式部署
 详见<http://hadoop.apache.org/docs/r1.2.1/cluster_setup.html>

---
参考文档：<http://hadoop.apache.org/docs/r1.2.1/single_node_setup.html>
