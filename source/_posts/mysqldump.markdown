---
layout: post
title: "mysqldump命令使用"
date: 2012-12-04 15:32
comments: true
categories: mysql
tags: [ mysql, dump ]
---
mysqldump命令用来备份数据库，默认会导出一整条insert语句，虽说执行起来会快一些。但是遇到大表，很可能因为缓冲区过载而挂掉。   
 
mysqldump --skip-opt 加入这个参数，就可以导出多条独立的insert语句。  
  
例如：
 
	mysqldump --skip-opt -uroot -p database tablename > script.sql