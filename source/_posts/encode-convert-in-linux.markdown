---
layout: post
title: "Linux中文件编码转换"
date: 2014-11-13 16:20
comments: true
categories: linux
tags: [ enca, iconv, convmv, 编码 ]
---
　在工作中，经常会遇到使用操作系统不一样的环境，从而导致在不同环境下的文件编辑的编码是不一样的，Windows默认是GBK编码格式，Linux默认是UTF-8的格式，这样就会出现把GBK编码的文件拷贝到Linux下出现乱码情况，很是让人头疼，下面给大家介绍下GBK->UTF-8文件编码批量转换。

Linux命令-enca 查看文件的编码

Enca语法

```shell
Usage:  enca [-L LANGUAGE] [OPTION]... [FILE]...
        enconv [-L LANGUAGE] [OPTION]... [FILE]...
        Detect encoding of text files and convert them if required.
```
Enca用法

```shell
$ enca -L zh_CN file 检查文件的编码
$ enca -L zh_CN -x UTF-8 file 将文件编码转换为"UTF-8"编码
$ enca -L zh_CN -x UTF-8 file1 file2 如果不想覆盖原文件可以这样
```
除了有检查文件编码的功能以外，”enca”还有一个好处就是如果文件本来就是你要转换的那种编码，它不会报错，还是会print出结果来， 而”iconv”则会报错。这对于脚本编写是比较方便的事情。
<!--more-->
转换单个文件的编码

```shell
$ enca -L none -x utf-8  index.html
```
转换多个文件的编码

```shell
$ enca -x utf-8 *
```
Linux文件名编码批量转换--convmv

Convmv语法

```shell
$ convmv -f 源编码 -t 新编码 [选项] 文件名
```
Convmv 常用参数

```shell
-r 递归处理子文件夹
–notest 真正进行操作，请注意在默认情况下是不对文件进行真实操作的，而只是试验。
–list 显示所有支持的编码
–unescap 可以做一下转义，比如把%20变成空格
```
示例

转换一个文件由GBK转换成UTF-8

```shell
convmv -f GBK -t UTF-8 --notest utf8 filename
```
GBK->UTF-8文件编码批量转换脚本

```shell
$ find default -type f -exec convmv -f GBK -t UTF-8 --notest utf8 {} -o utf/{} \;
```
使用iconv 转换

Iconv语法

```shell
iconv -f encoding -t encoding inputfile
```
示例

单个文件转换

```shell
$ iconv -f GBK -t UTF-8 file1 -o file2
```
批量转换

```shell
$ find default -type d -exec mkdir -p utf/{} \;
$ find default -type f -exec iconv -f GBK -t UTF-8 {} -o utf/{} \;
```
这两行命令将default目录下的文件由GBK编码转换为UTF-8编码，目录结构不变，转码后的文件保存在utf/default目录下。

---
原文:<http://blog.csdn.net/a280606790/article/details/8504133>