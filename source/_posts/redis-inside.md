---
title: Redis数据模型详解
date: 2018-12-25 10:47:52
tags: [ redis, ziplist, quicklist, sds ]
categories: redis
---

### Redis对象

在Redis的世界里，存储的所有值都是这个RedisObject对象。

源码定义如下：

```c
typedef struct redisObject {

    // 类型
    unsigned type:4;
    // 编码
    unsigned encoding:4;
    // 对象最后一次被访问的时间
    unsigned lru:REDIS_LRU_BITS; /* lru time (relative to server.lruclock) */
    // 引用计数
    int refcount;
    // 指向实际值的指针
    void *ptr;

} robj;
```

其中type可以取值枚举如下：

| TYPE枚举   | VALUE值 | 代表   |
| ---------- | ------- | ------ |
| OBJ_STRING | 0       | STRING |
| OBJ_LIST   | 1       | LIST   |
| OBJ_SET    | 2       | SET    |
| OBJ_ZSET   | 3       | ZSET   |
| OBJ_HASH   | 4       | HASH   |
| OBJ_MODULE | 5       | ---    |
| OBJ_STREAM | 6       | ---    |

不同类型Type，它的底层实现/编码方式也是不一样的。枚举类型如下：

| encoding枚举               | VALUE值 | str描述                |
| -------------------------- | ------- | ---------------------- |
| OBJ_ENCODING_RAW           | 0       | raw                    |
| OBJ_ENCODING_INT           | 1       | int                    |
| OBJ_ENCODING_HT            | 2       | hashtable              |
| OBJ_ENCODING_ZIPMAP        | 3       | 不再使用, 转为ZIPLIST  |
| OBJ_ENCODING_LINKEDLIST    | 4       | 不再使用,转为QUICKLIST |
| OBJ_ENCODING_ZIPLIST       | 5       | ziplist                |
| OBJ_ENCODING_INTSET        | 6       | intset                 |
| OBJ_ENCODING_SKIPLIST      | 7       | skiplist               |
| OBJ_ENCODING_EMBSTR        | 8       | embstr                 |
| OBJ_ENCODING_QUICKLIST     | 9       | quicklist              |
| define OBJ_ENCODING_STREAM | 10      | ---                    |



<!--more-->



### 数据类型结构

### String

Redis引入了一个SDS(Simple Dynamic String)类型，来表示String对象。

```c
struct sdshdr {
    
    // buf 中已占用空间的长度
    int len;
    // buf 中剩余可用空间的长度
    int free;
    // 数据空间
    char buf[];
};

```

自**Redis3.2**版本之后，为了更好的优化内存，把sdshdr分为sdshdr5、sdshdr8、sdshdr16、sdshdr32、sdrhdr64。

```c
/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

#### 字典类型

dictEntry表示hash表的节点。

```c
/*
 * 哈希表节点
 */
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;

} dictEntry;
```

dictht表示一个哈希表。
```c
/*
 * 哈希表
 *
 * 每个字典都使用两个哈希表，从而实现渐进式 rehash 。
 */
typedef struct dictht {
    
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;
    // 该哈希表已有节点的数量
    unsigned long used;

} dictht;
```

dict代表了一个字典，需要注意的是dictht有2个，这个用来实现渐进式rehash。

```c
/*
 * 字典
 */
typedef struct dict {

    // 类型特定函数
    dictType *type;
    // 私有数据
    void *privdata;
    // 哈希表
    dictht ht[2];
    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */
    // 目前正在运行的安全迭代器的数量
    int iterators; /* number of iterators currently running */

} dict;
```

相关结构图示如下：

![Redis 字典 dict](/images/redis-inside/redis-dict.png)



#### INTSET结构

用于保存INT类型集合的结构，很简单：

```c
typedef struct intset {
    
    // 编码方式
    uint32_t encoding;
    // 集合包含的元素数量
    uint32_t length;
    // 保存元素的数组
    int8_t contents[];

} intset;
```

内部维护了一个contents数组来保存具体的类型。



#### SKIPLIST(跳表)



```c
/* ZSETs use a specialized version of Skiplists */
/*
 * 跳跃表节点
 */
typedef struct zskiplistNode {

    // 成员对象
    robj *obj;
    // 分值
    double score;
    // 后退指针
    struct zskiplistNode *backward;
    // 层
    struct zskiplistLevel {
        // 前进指针
        struct zskiplistNode *forward;
        // 跨度
        unsigned int span;
    } level[];

} zskiplistNode;

/*
 * 跳跃表
 */
typedef struct zskiplist {

    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;
    // 表中节点的数量
    unsigned long length;
    // 表中层数最大的节点的层数
    int level;

} zskiplist;
```

图示表示为：

![redis skiplist 跳表](/images/redis-inside/redis-skiplist.png)



这个和常规意义上的跳表并没有区别。

#### LINKEDLIST

这是一个双向链表结构(adlist.h)。

```c
typedef struct listNode {

    // 前置节点
    struct listNode *prev;
    // 后置节点
    struct listNode *next;
    // 节点的值
    void *value;

} listNode;
```

具体的双向链表定义如下：

```c
/*
 * 双端链表结构
 */
typedef struct list {
    // 表头节点
    listNode *head;
    // 表尾节点
    listNode *tail;
    // 节点值复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void (*free)(void *ptr);
    // 节点值对比函数
    int (*match)(void *ptr, void *key);
    // 链表所包含的节点数量
    unsigned long len;

} list;
```

链表操作的具体实现在adlist.c里实现，同正常的双向链表没有区别。



#### ZIPLIST

什么时候使用**ZIPLIST**编码呢？如何实现的呢？



![Redis ZIPLIST格式](/images/redis-inside/redis-ziplist.png)

| 字段    | 含义                                                         |
| ------- | ------------------------------------------------------------ |
| zlbytes | 该字段是压缩链表的第一个字段，是无符号整型，占用4个字节。用于表示整个压缩链表占用的**字节数**（包括它自己）。 |
| zltail  | 无符号整型，占用4个字节。用于存储从压缩链表头部到最后一个entry**（不是尾元素zlend）**的偏移量，在快速跳转到链表尾部的场景使用。 |
| zllen   | 无符号整型，占用2个字节。用于存储压缩链表中包含的entry总数。 |
| zlend   | 特殊的entry，用来表示压缩链表的末尾。**占用一个字节**，值为255（0xFF）。 |

Entry部分：

一般来说，一个entry由prevlen，encoding，entry-data三个字段组成，但当entry是个很小的整数时，会根据编码省略掉entry-data字段。

**prevlen**表示前一个entry的长度。

- 当长度小于255字节的时候，用一个字节存储。
- 当长度大于等于255字节的时候，用5个字节来存储。第1个字节设置为255（0xFF）,后4个字节来存储前一个entry的长度。

**encoding**存储分为以下情况。

1.如果元素内容为**字符串**，encoding这样来表示。

| encoding值                                        | 可表示长度 | 存储     |
| ------------------------------------------------- | ---------- | -------- |
| 00xx xxxx                                         | 6bit       | ---      |
| 01xx xxxx xxxx xxxx                               | 14bit      | 大端存储 |
| 1000 0000 xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx | 32bit      | 大端存储 |

2.如果元素为整数，encoding这样表示：

| encoding  | 解释                                                         |
| --------- | ------------------------------------------------------------ |
| 1100 0000 | 表示数字占用后面2个字节                                      |
| 1101 0000 | 表示数字占用后面4个字节                                      |
| 1110 0000 | 表示数字占用后面8个字节                                      |
| 1111 0000 | 表示数字占用后面3个字节                                      |
| 1111 1110 | 表示数字占用后面1个字节                                      |
| 1111 1111 | 表示压缩链表中最后一个元素（特殊编码0xFF）。即zlend          |
| 1111 xxxx | 表示只用后4位表示0~12的整数，由于0000，1110跟1111三种已经被占用，也就是说这里的xxxx四位只能表示0001~1101，转换成十进制就是数字1~13，但是redis规定它用来表示0~12，因此当遇到这个编码时，我们需要取出后四位然后减1来得到正确的值。 |

源码文档里举了这样的一个包含"2"和“5”的ZIPLIST来说明：

```shell
 0  1  2  3    4  5  6  7    8  9    10 11   12 13   14
[0f 00 00 00] [0c 00 00 00] [02 00] [00 f3] [02 f6] [ff]
       |             |          |       |       |     |
    zlbytes        zltail    entries   "2"     "5"   zlend
```

前4字节，表示为0x0f, 说明总共占用15字节。

接下来的4个字节，表示为0x0c，说明最后一个entry（这里是“5”）的offset是12.

接下来的2个字节表示zllen总共有0x02=2个entry。

接下来的2个字节00，前面一个长度为0（目前是第1个entry），接下来0xf3(1111<u>0011</u>),  按上面的分析，这里3-1=2.

接下来的2个字节，02表示前一个entry长度为2， 0xf6（1111<u>0110</u>）代表6-1=5。

最后一个字节0xff代表结束，最后一个。

接下来，如果在“5”后面插入“hello world”

```shell
 0  1  2  3    4  5  6  7    8  9    10 11   12 13  14   15    16 17 18 19 20 21 22 23 24 25 26   27
[1b 00 00 00] [0e 00 00 00] [03 00] [00 f3] [02 f6] [02] [0b] [48 65 6c 6c 6f 20 57 6f 72 6c 64] [ff]
       |             |          |       |       |                          |                      |
    zlbytes        zltail    entries   "2"     "5"                      "hello world"            zlend
```

注意到zlbytes变成了28， zltail变成了14, entries变成了3。新加入的entry“hello word”,02表示前一个长度为2. 0x0b(00<u>00 1011</u>)表示后面有数据有11个字节。紧跟其后的就是11个字节的“hello world”ASCII码。

#### QUICKLIST

QUICKLIST是一个ziplist的双向链表，这个定义很准确。它综合了LINKEDLIST和ZIPLIST. 

```c
typedef struct quicklistNode {
    struct quicklistNode *prev;
    struct quicklistNode *next;
    unsigned char *zl;
    unsigned int sz;             /* ziplist size in bytes */
    unsigned int count : 16;     /* count of items in ziplist */
    unsigned int encoding : 2;   /* RAW==1 or LZF==2 */
    unsigned int container : 2;  /* NONE==1 or ZIPLIST==2 */
    unsigned int recompress : 1; /* was this node previous compressed? */
    unsigned int attempted_compress : 1; /* node can't compress; too small */
    unsigned int extra : 10; /* more bits to steal for future usage */
} quicklistNode;
```

其中：

prev,next：分别指向前一个和后一个node，典型的双链表。

zl：指向ziplist结构，如果启用了压缩，那么指向的就是quicklistLZF结构了。

sz：表示指向ziplist的总大小。即使被压缩，也仍旧是未压缩的大小。

count：表示ziplist里面数据项的个数。

encoding：表示是否压缩（2表示LZF，1为RAW）。

container: 表示使用什么类型存数据。目前（NONE=1, ZIPLIST=2）

recompress:  表明这个数据是否压缩过。

attempted_compress: 测试用

extra: 扩展字段。

```c
/* quicklistLZF is a 4+N byte struct holding 'sz' followed by 'compressed'.
 * 'sz' is byte length of 'compressed' field.
 * 'compressed' is LZF data with total (compressed) length 'sz'
 * NOTE: uncompressed length is stored in quicklistNode->sz.
 * When quicklistNode->zl is compressed, node->zl points to a quicklistLZF */
typedef struct quicklistLZF {
    unsigned int sz; /* LZF size in bytes*/
    char compressed[];
} quicklistLZF;
```

sz: LZF压缩后的大小。

compressed: 存放压缩后ZIPLIST的char数组。

```c
/* quicklist is a 40 byte struct (on 64-bit systems) describing a quicklist.
 * 'count' is the number of total entries.
 * 'len' is the number of quicklist nodes.
 * 'compress' is: -1 if compression disabled, otherwise it's the number
 *                of quicklistNodes to leave uncompressed at ends of quicklist.
 * 'fill' is the user-requested (or default) fill factor. */
typedef struct quicklist {
    quicklistNode *head;
    quicklistNode *tail;
    unsigned long count;        /* total count of all entries in all ziplists */
    unsigned long len;          /* number of quicklistNodes */
    int fill : 16;              /* fill factor for individual nodes */
    unsigned int compress : 16; /* depth of end nodes not to compress;0=off */
} quicklist;
```

head,tail：分别指向头、尾

count: 所有ziplist数据项总和。

len: quicklistNode的总个数

fill: 16bit，ziplist大小设置。存放`list-max-ziplist-size`参数的值

compress: 16bit, 压缩深度设置。存放`list-compress-depth`参数的值。

简略图示如下：

![Redis quicklist](/images/redis-inside/redis-quicklist.png)

上图中redis.conf中配置如下：

```properties
# 每个ziplist最大存多少个
list-max-ziplist-size 3  
# 代表头尾有2个不压缩(黄色部分)。默认是0，都不压缩。通常认为头尾是需要经常操作的
list-compress-depth 2  
```

quicklist总分利用了ziplist的压缩比高，规避了量大效率低的问题。redis 3.2之后，默认使用QUICKLIST来实现.

#### ZIPMAP

格式如下：

```
<zmlen><len>"foo"<len><free>"bar"<len>"hello"<len><free>"world"0XFF
```

zmlen: 1byte, 表示当前zipmap的长度。当长度>254的时候，这个不需要。总长度需要遍历拿到

len: 后面跟着值的长度。如果第1字节在0-253，代表是一个单字节产股。如果第1字节是254，表示后面有4字节表示具体长度。遇到255(0xFF)表示结尾。

free: 表示未用的字符串。这种情况存在于修改内容的时候，比如foo->hi，则有1个的空白。



### 数据编码一览

| 数据类型 | 一般情况编码                       | 少量数据编码   | 数据为整形 |
| -------- | ---------------------------------- | -------------- | ---------- |
| String   | RAW                                | EMBSTR         | INT        |
| List     | LINKEDLIST(3.2前)/QUICKLIST(3.2后) | ZIPLIST(3.2前) | ---        |
| Set      | HT                                 | ---            | INTSET     |
| Hash     | HT                                 | ZIPMAP         | ---        |
| ZSET     | SKIPLIST                           | ZIPLIST        | ---        |

#### String(OBJ_STRING)类型

**EMBSTR**(OBJ_ENCODING_EMBSTR)编码(长度小于<39/44使用这种类型):

```c
jemalloc chunk size = 64;
sizeof(robj)=16;
sizeof(sdshdr)=8;  // 3.2之前
sizeof(sdshdr8)=3; // 3.2之后
sizeof('\0')=1;
```
所以：

Redis3.2前，64-16-8-1=39。

Redis3.2之后，64-16-3-1=44.

长度在39/44以内的话，可以直接存放在连续内存，省去了多次分配。

**INT**(OBJ_ENCODING_INT): 如果字符串都是整数的时候，使用INT编码。

ptr指针直接代表字符串的值。实际上Redis启动后，会默认创建10000个RedisObject, 用于代表地1-10000的整形，这个大小是可以配置的。

如果以上都不满足, 使用**OBJ_ENCODING_RAW**编码，即SDS类型。

#### List(OBJ_LIST)类型

在Redis3.2之后，统一使用**OBJ_ENCODING_QUICKLIST**来实现。

### SET(OBJ_SET)类型

redis.conf配置文件里：

```properties
set-max-intset-entries  512
```

意思是如果整数集合的元素个数超过512，则转为HT编码。当然，如果插入的是字符串，那也会直接转码，无视这个限制的。

### HASH(OBJ_HASH)类型

redis.conf配置文件里：

```
hash-max-ziplist-value 64 // ziplist中最大能存放的值长度
hash-max-ziplist-entries 512 // ziplist中最多能存放的entry节点数量
```

这些阈值，用于决定什么情况下使用何种类型编码。

以上可以看出。entry个数小于等于512并且value长度小于等于64的话才使用ZIPLIST，超出则选择HASHTABLE.

### SortedSet(OBJ_ZSET)类型

有序集合定义为zset

```c
/*
 * 有序集合
 */
typedef struct zset {
    // 字典，键为成员，值为分值
    // 用于支持 O(1) 复杂度的按成员取分值操作
    dict *dict;
    // 跳跃表，按分值排序成员
    // 用于支持平均复杂度为 O(log N) 的按分值定位成员操作
    // 以及范围操作
    zskiplist *zsl;

} zset;
```

可以看出，里面由一个dict和一个zskiplist来实现。dict用来存储key-score对， zskiplist用于快速定位查找。


```c
#define OBJ_ZSET_MAX_ZIPLIST_ENTRIES 128    // ziplist中最多存放的节点数
#define OBJ_ZSET_MAX_ZIPLIST_VALUE 64       // ziplist中最大存放的数据长度
```

在entry个数<=128并且value长度<=64的时候，使用的是ziplist，否则使用SKIPLIST格式。

![Redis 有序集合 zset](/images/redis-inside/redis-zset.png)







### 总结

Redis为了更大程度的提升性能，压缩数据大小， 在内存模型和数据结构上做了很多努力。