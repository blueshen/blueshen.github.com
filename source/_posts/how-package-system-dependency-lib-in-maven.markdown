---
layout: post
title: "Maven如何打包本地依赖包"
date: 2013-11-20 19:04
comments: true
categories: maven
tags: [ maven, package, war ]
---
#### Maven如何依赖本地包？
有些依赖包在mavencentral上是没有的。那么如何在项目中使用呢？

        <dependency>
            <groupId>org.wltea.ik-analyzer</groupId>
            <artifactId>ik-analyzer</artifactId>
            <version>3.2.8</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/lib/ik-analyzer-3.2.8.jar</systemPath>
        </dependency>
这里可以指明scope是system,然后制定这个依赖包的systemPath就可以啦。这里依ik-analyzer为例的。

#### 如何将本地包打到war包内？
打war包，一般直接执行`mvn clean package`即可，但是默认的情况下是不能将scope=system的本地包打包的。这个时候就需要显式的指定啦。如下面这样，默认将lib下的所有jar文件打包到WEB-INF/lib下。当然也是可以打包其他的文件的，诸如xml,properties等的。

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <warName>${project.artifactId}</warName>
                    <webResources>
                        <resource>
                            <directory>lib/</directory>
                            <targetPath>WEB-INF/lib</targetPath>
                            <includes>
                                <include>**/*.jar</include>
                            </includes>
                        </resource>
                    </webResources>
                </configuration>
            </plugin>

