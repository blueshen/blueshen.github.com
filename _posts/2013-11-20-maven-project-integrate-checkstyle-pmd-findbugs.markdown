---
layout: post
title: "Maven项目集成CheckStyle,PMD,FindBugs进行静态代码扫描"
date: 2013-11-20 19:22
comments: true
categories: maven
tags: [ maven, checkstyle, pmd, findbugs, 静态代码扫描 ]
---

在pom.xml里添加以下maven插件配置：   

        <!-- 静态代码检查 -->
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-checkstyle-plugin</artifactId>
        <version>2.11</version>
        <configuration>
            <configLocation>checkstyle.xml</configLocation>
            <includeResources>false</includeResources>
            <failOnViolation>true</failOnViolation>
            <violationSeverity>info</violationSeverity>
            <maxAllowedViolations>0</maxAllowedViolations>
            <consoleOutput>true</consoleOutput>
            <encoding>UTF-8</encoding>
            <includes>
                **\/package\/**.java,**\/otherpackage\/**.java
            </includes>
        </configuration>
    <!--     <executions>
             <execution>
                 <goals>
                     <goal>check</goal>
                 </goals>
                 <phase>validate</phase>
             </execution>
         </executions>-->
    </plugin>
    
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-pmd-plugin</artifactId>
        <version>2.7.1</version>
        <configuration>
            <failurePriority>5</failurePriority>
            <failOnViolation>true</failOnViolation>
            <targetJdk>${jdk.version}</targetJdk>
            <verbose>true</verbose>
            <outputEncoding>UTF-8</outputEncoding>
            <rulesets>
                <ruleset>pmd.xml</ruleset>
            </rulesets>
            <includes>
                <include>**\/package\/**.java</include>
                <include>**\/otherpackage\/**.java</include>
            </includes>
        </configuration>
      <!--   <executions>
             <execution>
                 <phase>package</phase>
                 <goals>
                     <goal>check</goal>
                 </goals>
             </execution>
         </executions>-->
    </plugin>
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>findbugs-maven-plugin</artifactId>
        <version>2.5.2</version>
        <configuration>
            <onlyAnalyze>
                cn.shenyanchao.package.*,
                cn.shenyanchao.otherpackage.*,
            </onlyAnalyze>
            <includeFilterFile>findbugs.xml</includeFilterFile>
            <failOnError>true</failOnError>
            <outputEncoding>UTF-8</outputEncoding>
        </configuration>
     <!--   <executions>
            <execution>
                <phase>package</phase>
                <goals>
                    <goal>check</goal>
                </goals>
            </execution>
        </executions>-->
    </plugin>
            
这些配置集成了checkstyle,pmd,findbugs的插件。并指明了要使用的规则集合（checkstyle.xml,pmd.xml,findbugs.xml）。
<!--more-->  
#####那么能否指定只扫描特定的包或者文件呢？
上面checkstyle用的是：  

    <includes>
        **\/package\/**.java,**\/otherpackage\/**.java
    </includes>
     
pmd是：    

    <includes>
        <include>**\/package\/**.java</include>
        <include>**\/otherpackage\/**.java</include>
    </includes>

findbugs则使用的是：
     
    <onlyAnalyze>
        cn.shenyanchao.package.*,
        cn.shenyanchao.otherpackage.*,
    </onlyAnalyze>
      
   
#####如何在编译期间或打包期间执行检查？

如上所示的注释掉部分，添加就可以了：

    <executions>
         <execution>
             <phase>package</phase>
             <goals>
                 <goal>check</goal>
             </goals>
         </execution>
     </executions>
这里的意思是在mvn 执行打包package的时候进行check操作。因此如果check不通过，那么将不会编译打包成功。   


