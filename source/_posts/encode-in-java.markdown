---
layout: post
title: "Java字符编码及其使用详解"
date: 2012-10-23 18:58
comments: true
categories: encode Java
tags: [ Java, class, String, encode ]
---
##Java的编译存储##
Java是跨平台的一种语言，这一概念想必已经深入人心。Java是如何实现跨平台的呢?其中起到重大作用的便是Unicode编码。在使用IDE进行开发时，比如ECLIPSE,IDEA等，可以指定源文件（.java）的编码格式，此处的编码格式是指Java文件自身的编码。Java文件可以用各种编码进行存储，考虑到兼容中文字符，大多采用GBK,UTF-8,GB18030等编码格式。但是经过javac命令编译后，生成的.class文件毫无疑问都是Unicode编码。这样在class被加载进JVM后，所有的对象都是Unicode进行编码的，这确保了Java的跨平台特性。  
**简言之**  

.java(任意编码) ---> .class(Unicode) ---> JVM内（Unicode）   

<!--more-->

##是什么导致了乱码的出现？##
在JVM内，从class文件加载的源码全部以UNICODE编码。即使如`String str = "中国";`这样的语句，在JVM内存中仍然是unicode编码的。可是，程序本身难免牵涉到外部文件的读写、与数据库的交互等。这样就会造成很多非unicode编码的字符存在于JVM中，这也就是乱码出现的根本原因所在。

##Java中如何实现字符编码转换？##
String类提供了三个重要的函数：   

	getBytes(String charsetName)
	getBytes()
	new String(byte[],String charsetName)
`getBytes(String charset)`的作用是将字符串按照指定的charset编码，返回其字节方式的表示。具体来说，实现的是从unicode-->charset的转变。比如“中文”，在JVM内存储为“4e2d 6587”,如果设置charset为GBK,则被编码为“d6d0 cec4”。如果charset=UTF-8,那么结果是“e4 b8 ad e6 96 87”。如果charset=ISO-8859-1,由于无法编码，将返回“3f 3f”,这是两个问号。这是因为Unicode->ISO-8859-1不能完成这两个字符集之间的映射，因此使用时需要注意，这也为乱码提供了存在的可能性。    

`getBytes()`与上面这个功能一致，只不过charset是采用的系统默认的。系统默认的charset是什么呢？恐怕在不同的机器上有不同的charset。当前环境的默认charset可以通过`Charset.defaultCharset()`来查看，据个人测试，eclipse内是“UTF-8”,windows中文环境默认是“GBK”。因此当你写下getBytes()的时候，乱码的祸根已经种下，当程序在不同的环境运行时，结果可能就不一样，乱码就这样铺天盖地扑面而来了。哭吧！     
 
`new String(byte[],String charsetName)`的作用是将字节数组按照charset进行识别，最终转化为Unicode存储在JVM内。因此，如上面所说，如果byte[]是以UTF-8等编码存储的时候，如果按照ISO-8859-1这些不能映射的编码识别，仍旧出现乱码。   

因此，使用诸如`new String(str.getBytes("utf-8"), "gbk")`这种类似的转化时，需要谨慎，防止字符集的不可映射造成的乱码。   

**完美解决方案：**   
考虑到Java的编译存储，以及字符在JVM中的组织格式。其实需要自己完成的就是：   
任意编码的字符 --> unicode --->任意编码的目标字符
也就是在InputStream时，指定源字符编码格式，这样Java会自动转换为JVM内部的Unicode格式。而在OutputStream时，指定目标编码格式，Java会自动从Unicode转化为目标编码。Unicode是一个桥梁，而这种转换是不需认为控制的。
以OutputStream为例：  
 
	String str = "123中文";
	System.out.println(str);
	System.out.println("默认字符:" +Charset.defaultCharset());
	BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(new File("e://encode.txt")),"GBK"));
	writer.write(str);
	writer.close();
在eclipse内，源文件是以UTF-8进行编码的，默认字符也为UTF-8,运行后encode.txt,内容正确显示，编码格式正确。  

在windows cmd下进行javac,java进行执行。控制台输出“123 涓  .. ”等乱码。总之中文字符不能显示啊。查看encode.txt，内容倒是无乱码，编码格式怎么是UTF-8呢。我又一次纠结了？`System.out.println()`是不受环境影响的啊，它总能够以Unicode方式进行显示的啊。Java不至于这么弱吧。   
好吧，继续GOOGLE.BAIDU,找到了原因所在。javac 有这样一个参数`-encoding`来指定.java的编码格式。如果没有指定，那么编译时，认为编码格式为`Charset.defaultCharset()`,显然本例源码是以UTF-8进行编码的，直接javac是以GBK来读取的，那么在读到JVM成为UNICODE编码时，已经是乱码状态，不再是“123中文”了。`System.out.println()`只是按照实际情况进行了输出，因此在编译时，必须指定源文件的编码格式，保证了在Java文件内的常量字符的正常显示。  

	javac -encoding utf-8 *.java    
这也就可以理解，maven中pom.xml定义编码的原因了：   

	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
同样Ant的build.xml中，也有相应的编码配置选项：   

	<javac destdir="${build.dir}" encoding="UTF-8">
		<src path="${src.dir}" />
		<classpath refid="project.classpath" />
	</javac>
##HTTP的request与response乱码问题##
1.HTTP请求主要分为POST,GET两种情况。  
> POST的情况下，如何设置数据的编码格式呢。首先可以通过`<form accept-charset= "UTF-8">`来设置表单的编码格式。如果不设置，浏览器会直接使用网页的编码。JSP的网页编码一般通过`<%@page pageEncoding="UTF-8"%>`来指定JSP文件的**存储编码**。而通过`<%@ page contentType="text/html; charset=UTF-8" %>`来指定**输出内容编码**。`<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">`这一meta设置，用来指定**网页编码**，这一设置同样使用于静态页面。  

>GET情况下，参数都是放在URL内的，这时处理起来都比较麻烦。URL中的中文字符，一般都会进行`UrlEncode.encode()`处理,此时编码方式以系统默认编码为准，而在服务器端通过`getParameter`获得字符串是通过ISO-8859-1进行编码的。因此，需要从web服务器、浏览器端来同时考虑解决问题。服务器端的默认编码一般由`LC_ALL,LANG`决定的。通常可以设置为zh_CN.UTF-8。至于从浏览器端来解决，则没有统一的方法，毕竟环境多样。  

2.setCharacterEncoding()方法   
这一函数用来设置HTTP请求与相应的编码。前面提到过，通过`getParameter()`获得的字符串默认是以ISO-8859-1编码的。然而，如果使用了`request.setCharacterEncoding()`,则改变了其默认编码。同理，使用了`response.setCharacterEncoding`则保证了页面输出内容的编码，告诉浏览器输出内容采用什么编码。   
在spring的WEB项目中，有这样一个常用的filter：   

	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
这个filter，一看就知道是给编码有关，但是它具体做了哪些操作呢？源码来了：   

	protected void doFilterInternal(
			HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		if (this.encoding != null && (this.forceEncoding || request.getCharacterEncoding() == null)) {
			request.setCharacterEncoding(this.encoding);**
			if (this.forceEncoding) {
				response.setCharacterEncoding(this.encoding);
			}
		}
		filterChain.doFilter(request, response);
	}
可以看出，就是做了request、response的setCharacterEncoding()操作，防止乱码的问题出现。   
##数据库乱码##
前面讲到过，乱码的原因是JVM内可能会读到外界的未知编码字符，导致在内存中的Unicode编码为乱码。显然数据库就是其中之一，并且是经常遇到的问题。  
1.首先，数据库要支持多语言，应该考虑将数据库设置成UTF或者Unicode编码，而UTF更适合存储（UTF是变长存储的）。如果中文数据中包含的英文字符很少，建议Unicode。数据库编码设置，一般通过在配置文件设置`default-character-set=utf8`，这个在建库的时间也可以直接指定。   
2.读库的时候，可以在JDBC连接串中指定读取编码。如`useUnicode=true&characterEncoding=UTF-8`。并保证两者一致。		
##SecureCRT类似工具乱码##
这种问题，通常是由于SecureCRT等客户端与LINUX环境编码不一致造成的。同样的，在这些客户端读取数据库内容进行显示出现乱码，也是由于编码不一致的问题。   
可以通过设置客户端的编码，以及设置Linux环境的编码，使其保持一致来解决。

参考文档：<http://blog.csdn.net/qinysong/article/details/1179513>