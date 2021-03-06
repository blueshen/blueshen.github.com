---
layout: post
title: "ut-maven-plugin"
date: 2013-12-14 21:31
comments: true
categories: 单元测试
tags: [ unittest, ast, javaparser ]
---
### ut-maven-plugin简介

这是一个用来生成Unit Test模板的maven插件。使用这个插件不能彻底解决单元测试的问题，她还没有这么智能，只能按照自己的理解帮助你生成一个方法的单元测试方法框架。通过这些自动生成的代码，来提高写单元测试的生产率。
《程序员修炼之道》里的提示29说：“Write Code That Writes Code”。这也是ut-maven-plugin所做的，帮助程序员生成需要重复的工作以及共性的工作。

也许，现在她还小，还不足够智能，智能到足以测试你的方法的所有业务逻辑。但是在将来，她将会越来越智能。帮你解决更多的单元测试问题，或者解决更多共性的问题。
<!--more-->
### 实现原理
如何解析源码，这是首先需要解决的问题。在计算机科学中，有**抽象语法树(Abstract Syntax Tree)**这一概念，它是源代码的抽象语法结构的树状表现形式。树上的每个节点都表示源代码中的一种结构。利用抽象语法树就可以对源码进行一个全方位的解析，从而知道如何生成特定的测试代码。

Eclipse（以及其它IDE）中就提供了AST的解析功能，比如Eclipse里的outline(大纲)视图。

![](/images/blog/2013/eclipse-outline.png)

同时,Eclipse也提供的有抽象语法树视图，即ASTView。

![](/images/blog/2013/eclipse-ast-view.png)

本插件选用[JavaParser](https://code.google.com/p/javaparser/)来分析源码，提取并生成测试代码。

### 如何使用这个工具？
这个插件一个被我放到了[Maven Central](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22ut-maven-plugin%22)上，因此，你可以直接在pom.xml里添加上这个插件就可以了。同时建议你使用最新的版本。
比如：

```xml
<plugin>
<groupId>cn.shenyanchao.ut</groupId>
<artifactId>ut-maven-plugin</artifactId>
<version>0.2.9</version>
<executions>
    <execution>
        <id>source2test</id>
        <phase>process-test-sources</phase>
        <goals>
            <goal>source2test</goal>
        </goals>
    </execution>
</executions>
</plugin>
```

### 解决了什么问题？
以[spring-petclinic](https://github.com/spring-projects/spring-petclinic)中的代码为例。

下面的Service代码：

```java
package org.springframework.samples.petclinic.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.samples.petclinic.model.Pet;
import org.springframework.samples.petclinic.model.PetType;
import org.springframework.samples.petclinic.repository.PetRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.Collection;

@Service
public class ClinicServiceImpl implements ClinicService {

    @Autowired
    private PetRepository petRepository;
    //省略
    ......

    @Override
    @Transactional(readOnly = true)
    public Pet findPetById(int id) throws DataAccessException {
        return petRepository.findById(id);
    }
    //省略
    ......

}
```

那么，我们自己手工写的单元测试代码有可能是这样的：

```java
package org.springframework.samples.petclinic.service.test;

import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.springframework.samples.petclinic.model.Pet;
import org.springframework.samples.petclinic.repository.PetRepository;
import org.springframework.samples.petclinic.service.ClinicServiceImpl;

import static org.mockito.Matchers.anyInt;
import static org.mockito.Mockito.when;

public class ClinicServiceImplTest {

    @InjectMocks
    private ClinicServiceImpl clinicService = new ClinicServiceImpl();

    @Mock
    private PetRepository petRepository;

    @Before
    public void initMocks() {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void findPetByIdTest() {
        when(petRepository.findById(anyInt())).thenReturn(new Pet());
        Pet pet = clinicService.findPetById(1);
        Assert.assertNotNull(pet);
    }
```
如果，有很多个类需要写单元测试，那么我们会发现有很多代码是具有共性的，或者是有一定规律的。但从这个类来说，我们认为大部分代码都是可以通过对源代码进行分析得到的，除了以下的业务逻辑部分：

```java
        when(petRepository.findById(anyInt())).thenReturn(new Pet());
        Pet pet = clinicService.findPetById(1);
```
因此余下的代码都可以由插件来完成，使得程序员直接关注于业务逻辑部分的编写。大大的提高了程序员单元测试的编写效率，甚至使程序员们爱上单测。

当然，这里只是一个例子，如果能抽象出更多的共性，本插件就可以进行不断的扩展。简单的来说，有共性有规律就可以自动生成出来。随着不断的扩展，ut-maven-plugin将越来越智能化。

