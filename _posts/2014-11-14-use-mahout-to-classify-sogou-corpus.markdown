---
layout: post
title: "使用mahout对Sogou语料库进行分类"
date: 2014-11-14 13:26
comments: true
categories: mahout
tags: [ mahout, sogou, ik-analyzer, ubuntu ]
---
###软件版本
- Ubuntu Linux
- [mahout-0.9](http://mirror.bit.edu.cn/apache/mahout/0.9/mahout-distribution-0.9.tar.gz),本文写作的时候的最新版本
- [Sogou语料库](http://download.labs.sogou.com/dl/sogoulabdown/SogouC.reduced.20061102.tar.gz)精简版
- [ik-analyzer](https://github.com/blueshen/ik-analyzer), 这个版本是专门为了在mahout中进行分词而单独做的版本，源码从官方拿来。只更改了停用词，以及适配lucene4.6.1版本。maven化更方便使用。

###Sogou语料库处理
下载后的预料库，文档都是GB2312编码的。虽然mahout支持不同的编码方式，但是为了更方便的放到Hadoop里跑，还是建议先转化为标准的UTF-8.
语料库解压后，是sogou目录。我们执行以下代码进行转化，转换后的在utf/sogou目录下：

    find sogou -type d -exec mkdir -p utf/{} \;
    find sogou -type f -exec iconv -f GB2312 -t UTF-8 {} -o utf/{} \;

###使用mahout生成sequence file
进入utf/sogou目录，执行：

    mahout seqdirectory -i sogou -o sogou-seq -c UTF-8 -ow
生成的sequence file存放在sogou-seq目录内。
可以通过seqdumper命令查看：  
<!--more-->
    mahout seqdumper -i sogou-seq/part-m-00000 | more
![](/images/blog/2014/sogou-seqfile.png)

如果是在hadoop上跑，可以这样看。

    hadoop fs -text sogou-seq/part-m-00000 | more

###使用seq2sparse生成Vectors
执行命令：

    mahout seq2sparse -i sogou-seq  -o sogou-vectors -lnorm -nv -wt tfidf -a org.wltea.analyzer.lucene.IKAnalyzer -ow

查看生成的vector

mahout vectordump -i sogou-vectors/tfidf-vectors/part-r-00000 | more

![](/images/blog/2014/sogou-vector.png)

需要注意的是`org.wltea.analyzer.lucene.IKAnalyzer`，是上面提到的ik-analyzer里的。需要将ik-analyzer打包，然后将打出的包，放入$MAHOUT_HOME/lib内。默认是英文的，使用的是`org.apache.lucene.analysis.standard.StandardAnalyzer`，空格分割明显不适用中文。

###切分训练集和测试集

    mahout split -i sogou-vectors/tfidf-vectors/ --trainingOutput sogou-train-vectors --testOutput sogou-test-vectors --randomSelectionPct 40 --overwrite --sequenceFiles -xm sequential

###使用Native Bayes训练model

    mahout trainnb -i sogou-train-vectors -el -o sogou-model -li sogou-labelindex -ow -c

###使用测试集来查看效果

    mahout testnb -i sogou-test-vectors -m sogou-model -l sogou-labelindex -ow -o sogou-testing -c
![](/images/blog/2014/sogou-result.png)

可以看出87%的正确率还是不错的。

---
参考文档：

<http://mahout.apache.org/users/classification/twenty-newsgroups.html>     
<http://www.sogou.com/labs/dl/c.html>
