---
title: Redis-IO模型
date: 2018-12-26 15:18:49
tags: [ redis, io, epoll ]
categories: redis
---

Redis 是一个事件驱动的内存数据库，服务器需要处理两种类型的事件。

- 文件事件
- 时间事件

# 文件事件(FileEvent)

Redis 服务器通过 socket 实现与客户端（或其他redis服务器）的交互,文件事件就是服务器对 socket 操作的抽象。 Redis 服务器，通过监听这些 socket 产生的文件事件并处理这些事件，实现对客户端调用的响应。

## Reactor

Redis 基于 Reactor 模式开发了自己的事件处理器。

这里就先展开讲一讲 Reactor 模式。看下图：



![reactor](/images/redis-io/Reactor.jpg)



“I/O 多路复用模块”会监听多个 FD ，当这些FD产生，accept，read，write 或 close 的文件事件。会向“文件事件分发器（dispatcher）”传送事件。

文件事件分发器（dispatcher）在收到事件之后，会根据事件的类型将事件分发给对应的 handler。

我们顺着图，从上到下的逐一讲解 Redis 是怎么实现这个 Reactor 模型的。

<!--more-->

## I/O 多路复用模块

Redis 的 I/O 多路复用模块，其实是封装了操作系统提供的 select，epoll，avport 和 kqueue 这些基础函数。向上层提供了一个统一的接口，屏蔽了底层实现的细节。

| 操作系统 | I/O多路复用 |
| -------- | ----------- |
| Solaris  | avport      |
| LINUX    | epoll       |
| Mac      | kqueue      |
| Other    | select      |

下面以Linux epoll为例，看看使用 Redis 是怎么利用 linux 提供的 epoll 实现I/O 多路复用。

[LINUX epoll](https://www.shenyanchao.cn/blog/2018/12/25/epoll/)里介绍了epoll的3个方法。

Redis 对文件事件，封装epoll向上提供的接口：

```c
/*
 * 事件状态
 */
typedef struct aeApiState {
    // epoll_event 实例描述符
    int epfd;
    // 事件槽
    struct epoll_event *events;
} aeApiState;

/*
 * 创建一个新的 epoll 
 */
static int  aeApiCreate(aeEventLoop *eventLoop)
/*
 * 调整事件slot的大小
 */
static int  aeApiResize(aeEventLoop *eventLoop, int setsize)
/*
 * 释放epoll实例和事件slot
 */
static void aeApiFree(aeEventLoop *eventLoop)
/*
 * 关联给定事件到fd
 */
static int  aeApiAddEvent(aeEventLoop *eventLoop, int fd, int mask)
/*
 * 从fd中删除给定事件
 */
static void aeApiDelEvent(aeEventLoop *eventLoop, int fd, int mask)
/*
 * 获取可执行事件
 */
static int  aeApiPoll(aeEventLoop *eventLoop, struct timeval *tvp)
```

所以看看这个ae_peoll.c 如何对 epoll 进行封装的：

- `aeApiCreate()` 是对 `epoll.epoll_create()` 的封装。
- `aeApiAddEvent()`和`aeApiDelEvent()` 是对 `epoll.epoll_ctl()`的封装。
- `aeApiPoll()` 是对 `epoll_wait()`的封装。

这样 Redis 的利用 epoll 实现的 I/O 复用器就比较清晰了。

再往上一层次我们需要看看 ae.c 是怎么封装的？

首先需要关注的是事件处理器的数据结构：

```c
typedef struct aeFileEvent {

    // 监听事件类型掩码，
    // 值可以是 AE_READABLE 或 AE_WRITABLE ，
    // 或者 AE_READABLE | AE_WRITABLE
    int mask; /* one of AE_(READABLE|WRITABLE) */
    // 读事件处理器
    aeFileProc *rfileProc;
    // 写事件处理器
    aeFileProc *wfileProc;
    // 多路复用库的私有数据
    void *clientData;
} aeFileEvent;
```

`mask` 就是可以理解为事件的类型。

除了使用 ae_epoll.c 提供的方法外, ae.c 还增加 “增删查” 的几个 API。

- 增:`aeCreateFileEvent`
- 删:`aeDeleteFileEvent`
- 查: 查包括两个维度 `aeGetFileEvents` 获取某个 fd 的监听类型和`aeWait`等待某个fd 直到超时或者达到某个状态。

## 事件分发器（dispatcher）

Redis 的事件分发器 `ae.c/aeProcessEvents` 不但处理文件事件还处理时间事件，所以这里只贴与文件分发相关的出部分代码，dispather 根据 mask 调用不同的事件处理器。

```c
    //从 epoll 中获关注的事件
    numevents = aeApiPoll(eventLoop, tvp);
    for (j = 0; j < numevents; j++) {
        // 从已就绪数组中获取事件
        aeFileEvent *fe = &eventLoop->events[eventLoop->fired[j].fd];

        int mask = eventLoop->fired[j].mask;
        int fd = eventLoop->fired[j].fd;
        int rfired = 0;

        // 读事件
        if (fe->mask & mask & AE_READABLE) {
            // rfired 确保读/写事件只能执行其中一个
            rfired = 1;
            fe->rfileProc(eventLoop,fd,fe->clientData,mask);
        }
        // 写事件
        if (fe->mask & mask & AE_WRITABLE) {
            if (!rfired || fe->wfileProc != fe->rfileProc)
                fe->wfileProc(eventLoop,fd,fe->clientData,mask);
        }

        processed++;
    }
```

可以看到这个分发器，根据 mask 的不同将事件分别分发给了读事件和写事件。

## 文件事件处理器的类型

Redis 有大量的事件处理器类型，我们就讲解处理一个简单命令涉及到的3个处理器：

- acceptTcpHandler 连接应答处理器，负责处理连接相关的事件，当有client 连接到Redis的时候们就会产生 AE_READABLE 事件。引发它执行。
- readQueryFromClient 命令请求处理器，负责读取通过 sokect 发送来的命令。
- sendReplyToClient 命令回复处理器，当Redis处理完命令，就会产生 AE_WRITEABLE 事件，将数据回复给 client。

## 文件事件实现总结

我们按照开始给出的 Reactor 模型，从上到下讲解了文件事件处理器的实现，下面将会介绍时间时间的实现。

# 时间事件(TimeEvent)

Reids 有很多操作需要在给定的时间点进行处理，时间事件就是对这类定时任务的抽象。

先看时间事件的数据结构：

```c
/* Time event structure
 *
 * 时间事件结构
 */
typedef struct aeTimeEvent {

    // 时间事件的唯一标识符
    long long id; /* time event identifier. */

    // 事件的到达时间
    long when_sec; /* seconds */
    long when_ms; /* milliseconds */

    // 事件处理函数
    aeTimeProc *timeProc;

    // 事件释放函数
    aeEventFinalizerProc *finalizerProc;

    // 多路复用库的私有数据
    void *clientData;

    // 指向下个时间事件结构，形成链表
    struct aeTimeEvent *next;

} aeTimeEvent;
```

看见 `next` 我们就知道这个 aeTimeEvent 是一个链表结构。看图：



![timeEvent](/images/redis-io/timeEvent.jpg)



注意这是一个按照id**倒序排列**的链表，并没有按照事件顺序排序。

## processTimeEvent

Redis 使用这个函数处理所有的时间事件，我们整理一下执行思路：

1. 记录最新一次执行这个函数的时间，用于处理系统时间被修改产生的问题。
2. 遍历链表找出所有 when_sec 和 when_ms 小于现在时间的事件。
3. 执行事件对应的处理函数。
4. 检查事件类型，如果是周期事件则刷新该事件下一次的执行事件。
5. 否则从列表中删除事件。

# 综合调度器（aeProcessEvents）

综合调度器是 Redis 统一处理所有事件的地方。我们梳理一下这个函数的简单逻辑：

```c
// 1. 获取离当前时间最近的时间事件
shortest = aeSearchNearestTimer(eventLoop);
// 2. 获取间隔时间
timeval = shortest - nowTime;
// 如果timeval 小于 0，说明已经有需要执行的时间事件了。
if(timeval < 0){
    timeval = 0
}
// 3. 在 timeval 时间内，取出文件事件。
numevents = aeApiPoll(eventLoop, timeval);
// 4.根据文件事件的类型指定不同的文件处理器
if (AE_READABLE) {
    // 读事件
    rfileProc(eventLoop,fd,fe->clientData,mask);
}
    // 写事件
if (AE_WRITABLE) {
    wfileProc(eventLoop,fd,fe->clientData,mask);
}
```

以上的伪代码就是整个 Redis 事件处理器的逻辑。

我们可以再看看谁执行了这个 `aeProcessEvents`:

```c
void aeMain(aeEventLoop *eventLoop) {

    eventLoop->stop = 0;
    while (!eventLoop->stop) {
        // 如果有需要在事件处理前执行的函数，那么运行它
        if (eventLoop->beforesleep != NULL)
            eventLoop->beforesleep(eventLoop);

        // 开始处理事件
        aeProcessEvents(eventLoop, AE_ALL_EVENTS);
    }
}

```

然后我们再看看是谁调用了 `aeMain`:

```c
int main(int argc, char **argv) {
    //一些配置和准备
    ...
    aeMain(server.el);
    
    //结束后的回收工作
    ...
}
```

我们在 Redis 的 main 方法中找个了它。

这个时候我们整理出的思路就是:

- Redis 的 main() 方法执行了一些配置和准备以后就调用 `aeMain()` 方法。
- `eaMain()` while(true) 的调用 `aeProcessEvents()`。

所以我们说 Redis 是一个事件驱动的程序，期间我们发现，Redis 没有 fork 过任何线程。所以也可以说 Redis 是一个基于事件驱动的单线程应用。



---

参考文档：

[犀利豆的博客](https://www.xilidou.com/2018/03/22/redis-event/)