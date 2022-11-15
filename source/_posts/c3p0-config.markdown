---
layout: post
title: "关于C3P0容错和自动重连特性的研究"
date: 2015-03-26 13:07
comments: true
categories: 数据库
tags: [ c3p0, dhcp, 数据库]
---

最近常有数据库和网络设备升级和搬迁等事情，而各个应用都是基于数据库连接池做的，大部分都是基于C3P0，数据库或网络状况的变动都会导致客户端连接池中的connection失效，如何剔除这些blocked connection就和C3P0的各个配置息息相关。

C3P0容错和自动重连与以下配置参数有关：

`breakAfterAcquireFailure` ：true表示pool向数据库请求连接失败后标记整个pool为block并close，就算后端数据库恢复正常也不进行重连，客户端对pool的请求都拒绝掉。false表示不会标记 pool为block，新的请求都会尝试去数据库请求connection。默认为false。因此，如果想让数据库和网络故障恢复之后，pool能继续请求正常资源必须把此项配置设为false
<!--more-->
`idleConnectionTestPeriod` ：C3P0会有一个Task检测pool内的连接是否正常，此参数就是Task运行的频率。默认值为0，表示不进行检测。

`testConnectionOnCheckout` ：true表示在每次从pool内checkout连接的时候测试其有效性，这是个同步操作，因此应用端的每次数据库调用，都会先通过测试sql测试其有效性，如果连接无效，会关闭此连接并剔除出pool，并尝试从pool内取其他连接，默认为false，此特性要慎用，会造成至少多一倍的数据库调用。

`testConnectionOnCheckin` ：true表示每次把连接checkin到pool里的时候测试其有效性，因为是个事后操作，所以是异步的，应用端不需要等待测试结果，但同样会造成至少多一倍的数据库调用。

`acquireRetryAttempts 和acquireRetryDelay` ：pool请求取连接失败后重试的次数和重试的频率。请求连接会发生在pool内连接少于min值或则等待请求数>池内能提供的连接数

`automaticTestTable 、connectionTesterClassName 、preferredTestQuery` ：表示测试方式，默认是采用DatabaseMetaData.getTables()来测试connection的有效性，但可以通过以上配置来定制化测试语句，通过其名字就很好理解其含义，无需过多解释

`maxIdleTime 和 maxConnectionAge` ：表示connection的时效性，maxIdleTime和maxConnectionAge不同之处在于， maxIdleTime表示idle状态的connection能存活的最大时间，而 maxConnectionAge表示connection能存活的绝对时间

应用端getConnection抛出exception时， C3P0会测试其connection的有效性，并根据状态处理此connection，但应用端不会重调。

无论是网络问题还是远端数据库服务器，就算恢复正常后，客户端pool内其已存在的connection都会失效，要保证应用端调用无误，必须在checkout到应用端之前刷新这些无效connection

breakAfterAcquireFailure=false是关键。
如果 breakAfterAcquireFailure=true ，一旦pool向数据库请求连接失败，就会标记pool block并关闭pool，这样无论数据库是否恢复正常，应用端都无法从pool拿到连接

 要想保证网络和数据库瞬间的失效100%不会造成应用端getConnection失败必须开启testConnectionOnCheckout。但此特性的代价巨大，建议在应用端做容错。

推荐使用 idleConnectionTestPeriod。可以根据应用调用频率权衡一个检查pool的频率，这样可以在保证性能损耗不大情况下，尽可能的保证pool内connection的有效性

若嫌DatabaseMetaData.getTables()性能不好，可以尝试通过配置automaticTestTable、connectionTesterClassName、preferredTestQuery来找到一个性能最好的测试语句，只要能验证connection有效就行

综上所述，要想保证性能的前提下，本人推荐的配置组合如下：


```sh
breakAfterAcquireFailure: false
testConnectionOnCheckout: false
testConnectionOnCheckin: false
idleConnectionTestPeriod: 60
acquireRetryAttempts: 10
acquireRetryDelay: 1000
```


但需要注意的是以上的配置不能保证100%应用端getConnection无误，如果应用端不能发生getConnection错误，需要自行考虑容错和重试机制。

在以上配置下，当网络或数据库发生瞬间变动的情况下，会有如下事情发生：

- 自动测试idleConnection的 task轮训检测pool，对每个connction通过DatabaseMetaData.getTables()来测试有效性，并剔除无效连接。

- 根据请求情况和配置，pool向数据库请求新连接并加入池内

- 应用端getConnection->是否发生异常->如果发生异常，检验其有效性，并剔除出pool->如果没有发生异常（自动检查task之前已检测），调用成功
