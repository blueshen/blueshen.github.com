---
layout: post
title: "关于接口定义的一些想法"
description: ""
category: java
tags: [ spring, restful, interface]
---

最近公司内项目，需要开发接口给ios,android客户端去调用。因此，做了一些思考，主要是关于如何更好的定义接口以及联调。

### 使用RESTFul接口
RESTFul接口作为一个通行的标准，当然是优先使用的。项目都是基于Spring的，因此采用的是Spring MVC.


### 解决接口联调的苦难
作为一个技术人员，写文档是一大令人头疼的事情。况且即使把文档写好了，也可能导致在以后的修修补补中，导致不一致的情况。
因此，考虑采用文档自动生成的方式。最终选用了Swagger<wwww.swagger.io>来自动为Spring MVC生成文档。

- swagger-maven-plugin
- swagger-ui


### 项目中遇到的一些教训
作为技术负责人

- 提前制定接口，方便联调
- 提前规划返回接口类型，统一字段名称。