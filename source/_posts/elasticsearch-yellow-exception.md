---
title: ElasticSearch线上故障-持续Yellow状态
date: 2018-12-13 14:12:07
tags: [ elasticsearch, iperf ]
categories: [ elasticsearch]
---

### 现象

ES集群，状态持续Yellow。出现部分replica一直在追primary的索引数据，追不上。线上提供服务出现慢查询，导致499量增大。

固定的几个ES节点（10.10.24.X网段），出现间歇性被**踢出**Cluster，随后又**加入**Cluster。这是导致出现yellow原因，并且一直不能恢复到Green状态。由于Master节点在10.10.19.X网段, 感觉起来就是2个网段之间不通了。

<!--more-->

### 日志排查

```shell
 [2018-12-08T19:25:29,993][WARN ][o.e.x.s.t.n.SecurityNetty4Transport] [es35-search.mars.ljnode.com] write and flush on the network layer failed (channel: [id: 0x0e6ac038, L:0.0.0.0/0.0.0.0:8309 ! R:/10.200.24.96:46802])
java.nio.channels.ClosedChannelException: null
         at io.netty.channel.AbstractChannel$AbstractUnsafe.write(...)(Unknown Source) ~[?:?]
 [2018-12-08T19:25:29,994][WARN ][o.e.x.s.t.n.SecurityNetty4Transport] [es35-search.mars.ljnode.com] write and flush on the network layer failed (channel: [id: 0x0e6ac038, L:0.0.0.0/0.0.0.0:8309 ! R:/10.200.24.96:46802])
 java.nio.channels.ClosedChannelException: null
         at io.netty.channel.AbstractChannel$AbstractUnsafe.write(...)(Unknown Source) ~[?:?]
 [2018-12-08T19:25:31,111][DEBUG][o.e.a.a.c.n.i.TransportNodesInfoAction] [es35-search.mars.ljnode.com] failed to execute on node [_TS9PjOLRke8j12-d59iGA]
 org.elasticsearch.transport.ReceiveTimeoutTransportException: [es39-search.mars.ljnode.com_node0][10.200.24.86:8309][cluster:monitor/nodes/info[n]] request_id [203863696] timed out after [5000ms]
         at org.elasticsearch.transport.TransportService$TimeoutHandler.run(TransportService.java:961) [elasticsearch-5.6.9.jar:5.6.9]
         at org.elasticsearch.common.util.concurrent.ThreadContext$ContextPreservingRunnable.run(ThreadContext.java:575) [elasticsearch-5.6.9.jar:5.6.9]
         at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [?:1.8.0_172]
         at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [?:1.8.0_172]
         at java.lang.Thread.run(Thread.java:748) [?:1.8.0_172]
 [2018-12-08T19:25:32,305][DEBUG][o.e.a.a.c.n.i.TransportNodesInfoAction] [es35-search.mars.ljnode.com] failed to execute on node [yt3RGpgJStKwtOENBe5Trw]
 org.elasticsearch.transport.ReceiveTimeoutTransportException: [es38-search.mars.ljnode.com_node0][10.200.24.97:8309][cluster:monitor/nodes/info[n]] request_id [203863752] timed out after [5000ms]
         at org.elasticsearch.transport.TransportService$TimeoutHandler.run(TransportService.java:961) [elasticsearch-5.6.9.jar:5.6.9]
```

提示信息是底层的netty channel关闭，导致节点状态不可达，抛出ReceiveTimeoutTransportException，因此被踢出集群。

### 网络情况排查

以上的情况，怀疑连接有问题，可能存在脑裂。找运维同学排查，查看网络状况。

![机器吞吐](/images/elasticsearch-yellow-exception/elasticsearch-net-io.png)

通过监控图，首先可以排除网络断开。而整体的io吞吐，最高才只有150Mb(此处为bit)，网络流量也不大。



### 线上复现

由于线上的集群规模较大，复现集群搭建了3台ES，并分布在2个不同的网段。

那如何模拟高IO呢？

选择了[iperf](https://iperf.fr/)命令，来模拟网络流量，其实iperf主要是用来测试网络带宽以及丢包测试用的，也许有其他更好的工具。

具体使用方式参考[iperf命令](http://man.linuxde.net/iperf)。

果然在网络带宽被占满的时候，复现了上面的问题。但是当时线上并不高，这是什么原因。

在测试的时候，调试调用dstat命令，来观察真实的带宽。

![机器 dstat](/images/elasticsearch-yellow-exception/elasticsearch-machine-dstat.png)

可以看到，net带宽达到了117MB/s，已经把网络打满，这时我们在看运维的统计监控图：

![](/images/elasticsearch-yellow-exception/elasticsearch-net-io-1.png)

观察19:00左右的值，当时已经达到了117MB，但是统计出来值是很低的。和运维沟通后，确认采样是30s的平均值，所以在5~10秒短时间带宽打满，并不会体现出来。一切都可以解释了,happy。

### 原因&解决

网络是千兆的网卡，这在高并发，高吞吐的系统中，带宽100MB/s是不能满足需求的，瓶颈找到，那就更换为万兆网卡。

![elasticsearch network](/images/elasticsearch-yellow-exception/elasticsearch-network.png)

网络拓扑图如上，其中交换机和机器直连的网卡进行升级来解决。



### 有没有优化的空间

假如，机器硬件资源有限，有没有一些办法来尽可能规避。

ElasticSearch针对recovery有速度限制，默认是40MB。如果一台机器部署多台（实际上，我们就是这样部署的），那这个值可能是倍增的，很容易打满。
```
indices.recovery.max_bytes_per_sec: 10MB
```

减小上面的这个值，能够一定程度上缓解，由于recovery导致的持续带宽打满，并引起连锁反应。带宽满->节点掉->节点加入->recovery->带宽满->...



### 总结

网络带宽，应该在部署之前就进行充分的考虑和论证。根据不同的业务场景，选择不同的硬件配置。