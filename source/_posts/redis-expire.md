---
title: Redis过期&淘汰机制
date: 2018-12-26 14:28:52
tags: [ redis, expire, lru ]
categories: redis
---



## Redis内存限制

Redis作为一个高性能的内存NoSQL数据库，其容量受到最大内存限制的限制。

在使用Redis时，除了对性能，稳定性有很高的要求外，对内存占用也比较敏感。在使用过程中，有些用户会觉得自己的线上实例内存占用比自己预想的要大。

事实上，实例中的内存除了保存原始的键值对所需的开销外，还有一些运行时产生的额外内存，包括：

1. 垃圾数据和过期Key所占空间
2. 字典dict渐进式rehash导致未及时删除的空间
3. Redis管理数据, 包括底层数据结构开销，客户端信息，读写缓冲区等
4. 主从复制，bgsave时的额外开销
5. 其它

<!--more-->

## Redis过期数据清理策略

### 过期数据清理时机

为了防止一次性清理大量过期Key导致Redis服务受影响，Redis只在空闲时清理过期Key。

具体Redis逐出过期Key的时机为:

1. 访问Key时，会判断Key是否过期，逐出过期Key;

```c
  robj *lookupKeyRead(redisDb *db, robj *key) {
    robj *val;
    expireIfNeeded(db,key);
    val = lookupKey(db,key);
    ...
    return val;
}
```

1. CPU空闲时在定期serverCron任务中，逐出部分过期Key;

```c
        aeCreateTimeEvent(server.el, 1, serverCron, NULL, NULL)

        int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) {
            ...
            databasesCron();
            ...
        }
        void databasesCron(void) {
            /* Expire keys by random sampling. Not required for slaves
             + as master will synthesize DELs for us. */
            if (server.active_expire_enabled && server.masterhost == NULL)
                activeExpireCycle(ACTIVE_EXPIRE_CYCLE_SLOW);
            ...
        }
```

1. 每次事件循环执行的时候，逐出部分过期Key;

```c
 void aeMain(aeEventLoop *eventLoop) {
            eventLoop->stop = 0;
            while (!eventLoop->stop) {
                if (eventLoop->beforesleep != NULL)
                    eventLoop->beforesleep(eventLoop);
                aeProcessEvents(eventLoop, AE_ALL_EVENTS);
            }
        }

 void beforeSleep(struct aeEventLoop *eventLoop) {
            ...
   	/* Run a fast expire cycle (the called function will return
             - ASAP if a fast cycle is not needed). */
    if (server.active_expire_enabled && server.masterhost == NULL)
                activeExpireCycle(ACTIVE_EXPIRE_CYCLE_FAST);
            ...
 }
```

### 过期数据清理算法

Redis过期Key清理的机制对清理的频率和最大时间都有限制，在尽量不影响正常服务的情况下，进行过期Key的清理，以达到长时间服务的性能最优.

Redis会周期性的随机测试一批设置了过期时间的key并进行处理。测试到的已过期的key将被删除。具体的算法如下:

1. Redis配置项hz定义了serverCron任务的执行周期，默认为10，即CPU空闲时每秒执行10次;
2. 每次过期key清理的时间不超过CPU时间的25%，即若hz=1，则一次清理时间最大为250ms，若hz=10，则一次清理时间最大为25ms;
3. 清理时依次遍历所有的db;
4. 从db中随机取20个key，判断是否过期，若过期，则逐出;
5. 若有5个以上key过期，则重复步骤4，否则遍历下一个db;
6. 在清理过程中，若达到了25%CPU时间，退出清理过程;

这是一个基于概率的简单算法，基本的假设是抽出的样本能够代表整个key空间，redis持续清理过期的数据直至将要过期的key的百分比降到了25%以下。这也意味着在长期来看任何给定的时刻已经过期但仍占据着内存空间的key的量最多为每秒的写操作量除以4.

- 由于算法采用的随机取key判断是否过期的方式，故几乎不可能清理完所有的过期Key;
- 调高hz参数可以提升清理的频率，过期key可以更及时的被删除，但hz太高会增加CPU时间的消耗;[Redis作者关于hz参数的一些讨论](https://groups.google.com/forum/#!topic/redis-db/6kILekxQXBM)

代码分析如下:

```c
void activeExpireCycle(int type) {
    ...
    /* We can use at max ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC percentage of CPU time
     * per iteration. Since this function gets called with a frequency of
     * server.hz times per second, the following is the max amount of
     * microseconds we can spend in this function. */
    // 最多允许25%的CPU时间用于过期Key清理
    // 若hz=1，则一次activeExpireCycle最多只能执行250ms
    // 若hz=10，则一次activeExpireCycle最多只能执行25ms
    timelimit = 1000000*ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC/server.hz/100;
    ...
    // 遍历所有db
    for (j = 0; j < dbs_per_call; j++) {
        int expired;
        redisDb *db = server.db+(current_db % server.dbnum);

        /* Increment the DB now so we are sure if we run out of time
         * in the current DB we'll restart from the next. This allows to
         * distribute the time evenly across DBs. */
        current_db++;

        /* Continue to expire if at the end of the cycle more than 25%
         * of the keys were expired. */
        do {
            ...
            // 一次取20个Key，判断是否过期
            if (num > ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP)
                num = ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP;

            while (num--) {
                dictEntry *de;
                long long ttl;

                if ((de = dictGetRandomKey(db->expires)) == NULL) break;
                ttl = dictGetSignedIntegerVal(de)-now;
                if (activeExpireCycleTryExpire(db,de,now)) expired++;
            }

            if ((iteration & 0xf) == 0) { /* check once every 16 iterations. */
                long long elapsed = ustime()-start;
                latencyAddSampleIfNeeded("expire-cycle",elapsed/1000);
                if (elapsed > timelimit) timelimit_exit = 1;
            }
            if (timelimit_exit) return;
            /* We don't repeat the cycle if there are less than 25% of keys
             * found expired in the current DB. */
            // 若有5个以上过期Key，则继续直至时间超过25%的CPU时间
            // 若没有5个过期Key，则跳过。
        } while (expired > ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP/4);
    }
}
```

## Redis数据逐出策略

### 数据逐出时机

```c
// 执行命令
int processCommand(redisClient *c) {
        ...
        /* Handle the maxmemory directive.
        **
        First we try to free some memory if possible (if there are volatile
        * keys in the dataset). If there are not the only thing we can do
        * is returning an error. */
        if (server.maxmemory) {
            int retval = freeMemoryIfNeeded();
            ...
    }
    ...
}
```

### 数据逐出算法

在逐出算法中，根据用户设置的逐出策略，选出待逐出的key，直到当前内存小于最大内存值为主.

可选逐出策略如下：

- volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用 的数据淘汰
- volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数 据淘汰
- volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
- allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
- allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
- no-enviction（驱逐）：禁止驱逐数据

具体代码如下

```c
int freeMemoryIfNeeded() {
    ...
    // 计算mem_used
    mem_used = zmalloc_used_memory();
    ...

    /* Check if we are over the memory limit. */
    if (mem_used <= server.maxmemory) return REDIS_OK;

    // 如果禁止逐出，返回错误
    if (server.maxmemory_policy == REDIS_MAXMEMORY_NO_EVICTION)
        return REDIS_ERR; /* We need to free memory, but policy forbids. */

    mem_freed = 0;
    mem_tofree = mem_used - server.maxmemory;
    long long start = ustime();
    latencyStartMonitor(latency);
    while (mem_freed < mem_tofree) {
        int j, k, keys_freed = 0;

        for (j = 0; j < server.dbnum; j++) {
            // 根据逐出策略的不同，选出待逐出的数据
            long bestval = 0; /* just to prevent warning */
            sds bestkey = NULL;
            struct dictEntry *de;
            redisDb *db = server.db+j;
            dict *dict;

            if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_LRU ||
                server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_RANDOM)
            {
                dict = server.db[j].dict;
            } else {
                dict = server.db[j].expires;
            }
            if (dictSize(dict) == 0) continue;

            /* volatile-random and allkeys-random policy */
            if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_RANDOM ||
                server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_RANDOM)
            {
                de = dictGetRandomKey(dict);
                bestkey = dictGetKey(de);
            }

            /* volatile-lru and allkeys-lru policy */
            else if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_LRU ||
                server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_LRU)
            {
                for (k = 0; k < server.maxmemory_samples; k++) {
                    sds thiskey;
                    long thisval;
                    robj *o;

                    de = dictGetRandomKey(dict);
                    thiskey = dictGetKey(de);
                    /* When policy is volatile-lru we need an additional lookup
                     * to locate the real key, as dict is set to db->expires.  **/
                    if (server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_LRU)
                        de = dictFind(db->dict, thiskey);
                    o = dictGetVal(de);
                    thisval = estimateObjectIdleTime(o);

                    /* Higher idle time is better candidate for deletion */
                    if (bestkey == NULL || thisval > bestval) {
                        bestkey = thiskey;
                        bestval = thisval;
                    }
                }
            }

            /* volatile-ttl */
            else if (server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_TTL) {
                for (k = 0; k < server.maxmemory_samples; k++) {
                    sds thiskey;
                    long thisval;

                    de = dictGetRandomKey(dict);
                    thiskey = dictGetKey(de);
                    thisval = (long) dictGetVal(de);

                    /* Expire sooner (minor expire unix timestamp) is better
                     * candidate for deletion **/
                    if (bestkey == NULL || thisval < bestval) {
                        bestkey = thiskey;
                        bestval = thisval;
                    }
                }
            }

            /* Finally remove the selected key. **/
            // 逐出挑选出的数据
            if (bestkey ) {
                ...
                delta = (long long) zmalloc_used_memory();
                dbDelete(db,keyobj);
                delta -= (long long) zmalloc_used_memory();
                mem_freed += delta;
                ...
            }
        }
        ...
    }
    ...
    return REDIS_OK;
}
```

## 最佳实践

- 不要放垃圾数据，及时清理无用数据
  实验性的数据和下线的业务数据及时删除;
- key尽量都设置过期时间
  对具有时效性的key设置过期时间，通过redis自身的过期key清理策略来降低过期key对于内存的占用，同时也能够减少业务的麻烦，不需要定期手动清理了.
- 单Key不要过大
  key在get的时候网络传输延迟会比较大，需要分配的输出缓冲区也比较大，在定期清理的时候也容易造成比较高的延迟. 最好能通过业务拆分，数据压缩等方式避免这种过大的key的产生。
- 不同业务如果公用一个业务的话，最好使用不同的逻辑db分开
  从上面的分析可以看出，Redis的过期Key清理策略和强制淘汰策略都会遍历各个db。将key分布在不同的db有助于过期Key的及时清理。另外不同业务使用不同db也有助于问题排查和无用数据的及时下线.



---

参考文档

[Redis数据过期和淘汰策略详解](https://yq.aliyun.com/articles/257459)