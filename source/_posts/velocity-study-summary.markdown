---
layout: post
title: "Velocity学习小结"
date: 2014-03-31 16:24
comments: true
categories: velocity
tags: [ velocity, spring ]
---

### velocity在spring项目中的使用
本文，不是讲velocityResolver来渲染页面的只从最原始的使用方式如何使用。
首先，可以交给Spring来初始化velocityEngine:

```xml
<bean id="velocityEngine" class="org.springframework.ui.velocity.VelocityEngineFactoryBean">
    <property name="configLocation">
        <value>classpath:velocity.properties</value>
    </property>
    <property name="resourceLoaderPath">
        <value>/WEB-INF/velocity/</value>
    </property>
</bean>
```

其中的`configLocation`指明了velocity的配置文件路径。也就是说一些个性化的配置都可以直接在velocity.properties进行操作了。比如下面的例子：

```properties
resource.loader  =  file

file.resource.loader.description =  Velocity  File Resource Loader
file.resource.loader.class = org.apache.velocity.runtime.resource.loader.FileResourceLoader
file.resource.loader.cache =  true
file.resource.loader.modificationCheckInterval =  100

input.encoding = utf-8
output.encoding = utf-8
```

需要注意的是，resource.loader可能有多种选择，最常用的是file，class.当然也有webapp,jar等类型。file要求指明具体的路径，而在WEB应用里这块常常就会出现问题。因此，我们倾向于认为从classpath来加载模板。但是，为什么此处仍然推荐使用file而不是class呢。那是因为：

- spring增强了file加载的能力，推荐使用`resourceLoaderPath`来指明路径，而不是交给`file.resource.loader.path`进行处理。如果使用这个可能会跑NullPointerException；
- class加载存在弊端，在生产环境classpath里的内容一旦加载就被缓存起来了，这导致velocity模板加载的cache机制失效。


### 关于Velocity使用的坑

- 关于减号（-）的问题
>请注意下面的2中写法
`#set($maxIndex=$DOC_COUNT-1)`报错
`#set($maxIndex=$DOC_COUNT - 1)`正确，区别仅在于-两侧的空格

- 关于Range类型的问题
>`#set($array = [0..$maxIndex])` 这个里面$maxIndex应该只是一个变量，不能是一个表达式。比如这样`#set($array = [0..$maxIndex+1])`也是错误的。
