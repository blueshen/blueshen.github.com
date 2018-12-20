---
title: 利用MAT来分析JAVA内存泄露
date: 2018-10-29 14:53:46
categories: JVM
tags: [ java, mat, OOM]
---

### 如何DUMP出堆栈
#### 手动dump
	jmap -dump:format=b,file=<dumpfile.hprof> <pid>

#### JVM参数自动dump

	-XX:+HeapDumpOnOutOfMemoryError
	-XX:HeapDumpPath=${heap.dump.path}


### 下载并调整MAT(Eclipse Memory Analyze Tool)
下载地址： <https://www.eclipse.org/mat/downloads.php>

![mat](/images/blog/mat/mat-for-java.jpg)

依据不同的操作系统下载响应版本。

<!--more-->
### 解析dump出的文件
通常情况下，dump出的文件是很大的。需要修改一下MemoryAnalyzer.ini，调大`-Xmx`参数,至少要比要分析的文件相等。

```shell
-startup
../Eclipse/plugins/org.eclipse.equinox.launcher_1.5.0.v20180512-1130.jar
--launcher.library
../Eclipse/plugins/org.eclipse.equinox.launcher.cocoa.macosx.x86_64_1.1.700.v20180518-1200
-vmargs
-Xmx10G
-Dorg.eclipse.swt.internal.carbon.smallFonts
-XstartOnFirstThread
```

### 无界面执行（LINUX）

```shell
./ParseHeapDump.sh ${dump.prof} org.eclipse.mat.api:suspects
```

还支持另外两个：

org.eclipse.mat.api:overview
org.eclipse.mat.api:top_components

执行之后，产生多个zip版html。不过这个版本，没有直接分析出来的好用，有些功能有缺失。

---

其他线上问题常见分析工具：

<https://github.com/blueshen/useful-scripts>
