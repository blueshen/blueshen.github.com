---
layout: post
title: "从Github发布jar包到Maven Central"
date: 2013-06-19 19:50
comments: true
categories: maven
tags: [ maven, github, jar, sonatype ]
---

除非你的项目是Apache或者Codehaus管理的，否则你是不可能直接把artifacts发布到Maven Central的。然而，Sonatype提供了它们的Nexus repositories,我们可以将开源项目提交上去，然后会自动同步到Maven Central。

下面介绍下如何做：

### 准备工作

1.添加source code management信息到pom.xml:

```xml
<scm>
    <connection>scm:git:git@github.com:blueshen/ut-maven-plugin.git</connection>
    <developerConnection>scm:git:git@github.com:blueshen/ut-maven-plugin.git</developerConnection>
    <url>git@github.com:blueshen/ut-maven-plugin.git</url>
</scm>
```
2.创建GPG的密钥对并发布公钥。参看[Sonatype documentation](https://docs.sonatype.org/display/Repository/How+To+Generate+PGP+Signatures+With+Maven)的具体步骤。推荐使用Linux,Mac来发布。

>`gpg --gen-key` 创建key
`gpg --list-keys` 查看所有的key
`gpg --send-keys --keyserver pool.sks-keyservers.net yourkey`   发布你的key到常见服务器上，如果出现找不到key的话，多发几个服务器

3.确保你的工程POM符合[要求](https://docs.sonatype.org/display/Repository/Central+Sync+Requirements)
4.创建一个[Sonatype JIRA](https://issues.sonatype.org/)账户，并发布一个ticket来让Nexus repository建立。用户名，密码后面要用的。这中间牵涉到人工操作，会花费一些时间。
<!--more-->

### 使用Maven来发布到Sonatype Nexus repository
#### pom.xml配置有2种方法：

1.添加maven-release-plugin到pom.xml:


```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-release-plugin</artifactId>
	<version>2.2.2</version>
	<configuration>
		<arguments>-Dgpg.passphrase=${gpg.passphrase}</arguments>
	</configuration>
</plugin>
```

添加Sonatype repositories:

```xml
<distributionManagement>
	<snapshotRepository>
		<id>sonatype-nexus-snapshots</id>
		<name>Sonatype Nexus snapshot repository</name>
		<url>https://oss.sonatype.org/content/repositories/snapshots</url>
	</snapshotRepository>
	<repository>
		<id>sonatype-nexus-staging</id>
		<name>Sonatype Nexus release repository</name>
		<url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
	</repository>
</distributionManagement>
```

设置artifact signing:

```xml
<profiles>
	<profile>
		<id>release-sign-artifacts</id>
		<activation>
			<property>
				<name>performRelease</name>
				<value>true</value>
			</property>
		</activation>
		<build>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-gpg-plugin</artifactId>
					<version>1.4</version>
					<configuration>
						<passphrase>${gpg.passphrase}</passphrase>
					</configuration>
					<executions>
						<execution>
							<id>sign-artifacts</id>
							<phase>verify</phase>
							<goals>
								<goal>sign</goal>
							</goals>
						</execution>
					</executions>
				</plugin>
			</plugins>
		</build>
	</profile>
</profiles>
```
2.直接添加sonatype parent到pom.xml

```xml
<parent>
    <groupId>org.sonatype.oss</groupId>
    <artifactId>oss-parent</artifactId>
    <version>7</version>
</parent>
```

#### maven settings.xml配置
编辑或者创建~/.m2/settings.xml并包含验证信息:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
	<servers>
		<server>
			<id>sonatype-nexus-snapshots</id>
			<username>myusername</username>
			<password>mypassword</password>
		</server>
		<server>
			<id>sonatype-nexus-staging</id>
			<username>myusername</username>
			<password>mypassword</password>
		</server>
	</servers>

	<profiles>
		<profile>
			<id>sign</id>
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
			<properties>
				<gpg.passphrase>mypassphrase</gpg.passphrase>
			</properties>
		</profile>
	</profiles>
</settings>
```

Maven有一个[避免使用明文的方法](http://maven.apache.org/guides/mini/guide-encryption.html)。

### 准备一个release版本

为了准备一个release版本, 执行:

    $ mvn release:clean
    $ mvn release:prepare
[参考这个](http://maven.apache.org/plugins/maven-release-plugin/examples/prepare-release.html).这里具体做了以下工作：如果你的工程的版本是0.1-SNAPSHOT. 准备一个发布版本会去掉-SNAPSHOT后缀，然后提交到github，并将这时的代码打一个tag。同时，更新本地项目到0.2-SNAPSHOT版本.

如果你要撤销release,可以使用`git reset --hard HEAD~2`进行[回退](http://stackoverflow.com/a/6866485/150884),使用`git tag -d ut-maven-plugin-0.1`[删除Tag](http://nathanhoad.net/how-to-delete-a-remote-git-tag),然后使用`git push origin :refs/tags/ut-maven-plugin-0.1`提交.

### 发布到Sonatype

1.如果一切OK，你就可以使用[mvn release:perform](http://maven.apache.org/plugins/maven-release-plugin/examples/perform-release.html)来发布工程到Sonatype。
2.登录到Sonatype Nexus，在Staging Repositories找到你的artifacts。
3.点击close,关闭后，点击Release发布artifacts。Sonatype有一些[很好的指引](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-8.ReleaseIt). 你可以使用[Maven repository management plugin](http://www.sonatype.com/books/nexus-book/reference/staging-sect-managing-plugin.html)来自动化这些步骤,尽管我自己还没有试过。
4.在你的JIRA ticket下添加一个评论，说你已经推了release版本。下次Sonatype同步的时候，就会将你的artifacts放到Maven Central了。以后再发布新版本的时候，就不用添加评论了，会自动同步的。

5.一旦你的jira申请已经OK，如果你再发顶级目录的其他artifact也是不需要再走jira流程了。

参考文档<http://datumedge.blogspot.jp/2012/05/publishing-from-github-to-maven-central.html>
