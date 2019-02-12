---
title: Lucene数字类型处理
date: 2018-12-04 12:35:48
tags: [ lucene, numberic ]
categories: [ lucene ]
---

### 前言

由于倒排索引的特殊性，在lucene搜索引擎里，一切都是**面向字符**的，因此数字、日期等类型是如何快速查找、区间匹配、排序的呢？下面将揭开Lucene数字类型处理的神秘面纱。

###  数值类型特点

#### 排序

以[123，123456，222]这3个数字为例，它们是按字母序排列的，但是先后顺序显然不是我们想要的。我们期待的排序结果是[123，222，123456]。

<!--more-->

如何来解决这个问题呢，最直接想到的就是变为定长的字符。转化为6位的定长字符:[000123，123456，000222]。这样我们就可以按字母序排序成[000123,000222,123456]了，符合预期。转化为**定长**是一个很好的思路。但是，也有一些问题，比如定长的长度，取决于最大数值的长度，为了通用，这个定长可能是相当长的，比如64。这会带来一定程度空间和效率浪费。

#### 区间查询

如果想查询符合某个取值范围的所有记录，比如查询价格在25到1000之间的所有商品。

由于索引面向的是字符，如果要查询在符合条件的商品，那需要枚举出所有的可能。类似于这样的查询(price=25) OR (price=26) OR (...) OR (price=999) OR (price=1000)。这个查询是相当低效的，这个例子的区间还是比较小的，加入是查询1到10000000呢，这在实现上是不可能的。另外如果查询条件太多，lucene本身会报TooManyClauseError。



### Lucene数值-TRIE

Lucene针对数值类型，专门提出了一种TRIE树结构。TrieInt，TrieLong，TrieDouble...

那么它究竟是怎么来处理数值类型的呢？

下面以int类型为例。找到类`NumbericUtils`,

```java
public static void intToPrefixCoded(final int val, final int shift, final BytesRefBuilder bytes) {
  // ensure shift is 0..31
  if ((shift & ~0x1f) != 0) {
    throw new IllegalArgumentException("Illegal shift value, must be 0..31; got shift=" + shift);
  }
  int nChars = (((31-shift)*37)>>8) + 1;    // i/7 is the same as (i*37)>>8 for i in 0..63
  bytes.setLength(nChars+1);   // one extra for the byte that contains the shift info
  bytes.grow(LegacyNumericUtils.BUF_SIZE_LONG);  // use the max
  bytes.setByteAt(0, (byte)(SHIFT_START_INT + shift)); //SHIFT_START_INT=60
  int sortableBits = val ^ 0x80000000;
  sortableBits >>>= shift;
  while (nChars > 0) {
    // Store 7 bits per byte for compatibility
    // with UTF-8 encoding of terms
    bytes.setByteAt(nChars--, (byte)(sortableBits & 0x7f));
    sortableBits >>>= 7;
  }
}
```

下面以250314这个数字为例，解析下上面的代码到底处理了什么。

![TRIE INT](/images/lucene-numberic/trieint.png)

通过以上的变换，保证了以下：

- 符号位取反，保证了顺序性
- 7bit切分，保证一个字节char可以表示
- 定长6个字节，任何整数都可以表示

整型数字转换为`0x6008000F234A`的16进制表示，**有序**和**定长**都得到了很好的解决。

那么，如何来解决**区间查询**的效率呢？

Solr在对数值型进行索引的时候，构建了一下的TRIE树结构，这也是为什么叫TrieInt的原因。

![](/images/lucene-numberic/solr-trie.png)

上图，如果想查询大于423小于642的区间，那不不用穷举出所有从423、424、... 641、642的所有可能性，毕竟这太低效了。

当然从上一节，我们知道转换后的字符串是一个16进制的，这个图中为了方便理解，统一为10进制。

在索引的时候，做了前缀的索引，正如方法名`intToPrefixCoded`所暗示。比如423，在索引的时候变成了4、42、423; 同理521变成了5、52、521。

当我们要查询符合[423,642]区间的数据时，我们只需要遍历TRIE树，只匹配存在图中黑色加粗标识的几个数字即可。图中即为423，44，5，63，641，642。这样查询的数量级大大的降低，也算是典型的空间换时间。



### Lucene Range查询

TODO