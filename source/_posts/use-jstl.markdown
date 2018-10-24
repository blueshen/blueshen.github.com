---
layout: post
title: "JSTL介绍"
date: 2014-03-18 18:49
comments: true
categories: java
tags: [ jstl, el ]
---
### EL 表达式
在JSP页面中，使用标签库代替传统的Java片段语言来实现页面的显示逻辑已经不是新技术了，然而，由自定义标签很容易造成重复定义和非标准的实现。鉴于此，出现了JSTL（JSP Standard Tag Library），为大多数JSP页面逻辑提供了实现的JSTL技术，该技术本身就是一个标签库。

Sun公司Java规范标准的JSTL由apache jakarta组织负责维护。作为开源的标准技术，它一直在不断地完善。JSTL的发布包有两个版本：Standard-1.0 Taglib、Standard-1.1 Taglib，它们在使用时是不同的。

Standard-1.0 Taglib（JSTL1.0）支持Servlet2.3和JSP1.2规范，Web应用服务器Tomcat4支持这些规范，而它的发布也在Tomcat 4.1.24测试通过了。

Standard-1.1 Taglib（JSTL1.1）支持Servlet2.4和JSP2.0规范，Web应用服务器Tomcat5支持这些规范，它的发布在Tomcat 5.0.3测试通过了。

本文将以由Sun发布的Standard-1.1 Taglib标签库为主，而apache jakarta组织发布的开源标签库，可以从http://jakarta.apache.org/taglibs/找到所需要的帮助。Sun发布的标准JSTL1.1标签库有以下几个标签：

 - 核心标签库：包含Web应用的常见工作，比如：循环、表达式赋值、基本输入输出等。
 - 国际化标签库：用来格式化显示数据的工作，比如：对不同区域的日期格式化等。
 - 数据库标签库：可以做访问数据库的工作。
 - XML标签库：用来访问XML文件的工作，这是JSTL标签库的一个特点。
 - 函数标签库：用来读取已经定义的某个函数。

此外，JSTL还提供了EL表达式语言（Expression Language）来进行辅助的工作。

JSTL标签库由标签库和EL表达式语言两个部分组成。EL在JSTL 1.0规范中被引入，当时用来作为Java表达式来工作，而该表达式必须配合JSTL的标签库才能得到需要的结果。

说明：在JSTL 1.1规范中，JSP2.0容器已经能够独立的理解任何EL表达式。EL可以独立出现在JSP页面的任何角落。本文随后的内容将以JSTL 1.1规范作为介绍的重点。

<!--more-->

#### 9.2.1JSTL EL 表达式语言简介
EL是从JavaScript脚本语言得到启发的一种表达式语言，它借鉴了JavaScript多类型转换无关性的特点。在使用EL从scope中得到参数时可以自动转换类型，因此对于类型的限制更加宽松。Web服务器对于request请求参数通常会以String类型来发送，在得到时使用的Java语言脚本就应该是request.getParameter(“XXX”)，这样的话，对于实际应用还必须进行强制类型转换。而EL就将用户从这种类型转换的繁琐工作脱离出来，允许用户直接使用EL表达式取得的值，而不用关心它是什么类型。

下面的示例就是一个EL表达式，见例9.1。

例9.1：简单EL表达式

      <%@ page contentType="text/html; charset=UTF-8"%>
      <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
      <html>
       <body> ${sampleValue + 1} <br> </body>
      </html>

这个示例将在JSP页面显示为“1”，EL表达式必须以“${XXX}”来表示，其中“XXX”部分就是具体表达式内容，“${}”将这个表达式内容包含在其中作为EL表达式的定义。本示例可以在满足JSP2.0规范的任何Web应用服务器中使用。

#### 9.2.2 EL 表达式的默认变量
一个EL表达式包含变量和操作符两个内容。任何存在于JSP作用范围的JavaBean都可以被转化成EL表达式来使用，它所包含的默认变量如下：

1．默认变量pageScope、requestScope、sessionScope、applicationScope
      这4个默认变量包含Scope作用范围的参数集合，相当于被保存在java.util.Map中的某个参数。下面看简单的示例9.2：

例9.2：使用sessionScope变量的EL表达式

    <%request.getSession().setAttribute("sampleValue", new Integer(10));%>
    ${sessionScope.sampleValue}

取得保存在Session中参数的sessionScope变量的EL表达式，“.”是property访问操作符，在这里表示从Session中取得“键”为“sampleValue”的参数，并显示出来。显示结果为“10”。

2．默认变量param、paramValues
      这两个默认变量包含请求参数的集合，param表明请求包含的参数为单一控件，paramValues表明请求包含的参数为控件数组。下面看一个简单示例9.3：

例9.3：提交请求的页面和接受的页面

    <%@ page contentType="text/html; charset=UTF-8"%>
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
    <html>
     <body>
         <form action="SampleJsp.jsp">
        <input type="text" name="sampleValue" value="10">
        <input type="text" name="sampleValue" value="11">
        <input type="text" name="sampleValue" value="12">
        <input type="text" name="sampleSingleValue" value="SingleValue">
        <input type="submit" value="Submit">
        </form>
     </body>
    </html>

在这个页面中定义了两组控件，控件名为“sampleValue”的是一套控件数组，控件名为“sampleSingleValue”的是单一控件，通过递交将请求参数传送到SampleJsp.jsp。

    <%@ page contentType="text/html; charset=UTF-8"%>
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
    <html>
     <body>
        ${paramValues.sampleValue[2]} <br>
        ${param.sampleSingleValue} <br>
     </body>
    </html>

这是请求转发到的页面，通过EL表达式的paramValues变量得到控件数组中最后一个控件的递交参数，通过EL表达式的param变量得到单一控件的递交参数。控件数组参数的EL表达式使用“[]”来指定数组下标。本示例将显示控件数组中最后一个控件的值“12”和单一控件的值“SingleValue”。

3．默认变量header、headerValues

这两个默认变量包含请求参数头部信息的集合，header变量表示单一头部信息，headerValues则表示数组型的头部信息。

4．默认变量cookie

包含所有请求的cookie集合，集合中的每个对象对应javax.servlet.http.Cookie。

5．默认变量initParam

包含所有应用程序初始化参数的集合。

6．默认变量pageContext

等价于page环境类javax.servlet.jsp.PageContext的实例，用来提供访问不同的请求参数。
11个默认变量几乎包含了Web应用的所有基本操作，若一个表达式不使用这些变量而直接使用参数名，那么就采用就近原则。该表达式将使用最近取得的参数值。


### 表达式的操作符
EL表达式中还有许多操作符可以帮助完成各种所需的操作，之前的示例中“.”、“[]”就是其中的两个，下面将用表9.1来展示所有操作符及它们各自的功能。

表9.1 EL表达式的操作符

<table cellspacing="0" cellpadding="0" width="562" border="1">
						<tbody>
							<tr>
								<td width="64">
								<p align="left">操作符</p>
								</td>
								<td width="498">
								<p align="left">功能和作用</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">.</p>
								</td>
								<td width="498">
								<p align="left">访问一个bean属性或者Map entry</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">[]</p>
								</td>
								<td width="498">
								<p align="left">访问一个数组或者链表元素</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">()</p>
								</td>
								<td width="498">
								<p align="left">对子表达式分组，用来改变赋值顺序</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">? :</p>
								</td>
								<td width="498">
								<p align="left">条件语句，比如：条件?ifTrue:ifFalse</p>
								<p align="left">如果条件为真，表达式值为前者，反之为后者</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">+</p>
								</td>
								<td width="498">
								<p align="left">数学运算符，加操作</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">-</p>
								</td>
								<td width="498">
								<p align="left">数学运算符，减操作或者对一个值取反</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">*</p>
								</td>
								<td width="498">
								<p align="left">数学运算符，乘操作</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">/ 或div</p>
								</td>
								<td width="498">
								<p align="left">数学运算符，除操作</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">% 或mod</p>
								</td>
								<td width="498">
								<p align="left">数学运算符，模操作(取余)</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">== 或eq</p>
								</td>
								<td width="498">
								<p align="left">逻辑运算符，判断符号左右两端是否相等，如果相等返回true，否则返回false</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">!= 或ne</p>
								</td>
								<td width="498">
								<p align="left">逻辑运算符，判断符号左右两端是否不相等，如果不相等返回true，否则返回false</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">&lt; 或lt</p>
								</td>
								<td width="498">
								<p align="left">逻辑运算符，判断符号左边是否小于右边，如果小于返回true，否则返回false</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">&gt; 或gt</p>
								</td>
								<td width="498">
								<p align="left">逻辑运算符，判断符号左边是否大于右边，如果大于返回true，否则返回false</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">&lt;= 或le</p>
								</td>
								<td width="498">
								<p align="left">逻辑运算符，判断符号左边是否小于或者等于右边，如果小于或者等于返回true，否则返回false</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">&gt;= 或ge</p>
								</td>
								<td width="498">
								<p align="left">逻辑运算符，判断符号左边是否大于或者等于右边，如果大于或者等于返回true，否则返回false</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">&amp;&amp; 或and</p>
								</td>
								<td width="498">
								<p align="left">逻辑运算符，与操作赋。如果左右两边同为true返回true，否则返回false</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">|| 或or</p>
								</td>
								<td width="498">
								<p align="left">逻辑运算符，或操作赋。如果左右两边有任何一边为true返回true，否则返回false</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">! 或not</p>
								</td>
								<td width="498">
								<p align="left">逻辑运算符，非操作赋。如果对true取运算返回false，否则返回true</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">empty</p>
								</td>
								<td width="498">
								<p align="left">用来对一个空变量值进行判断: null、一个空String、空数组、空Map、没有条目的Collection集合</p>
								</td>
							</tr>
							<tr>
								<td valign="top" width="64">
								<p align="left">func(args)</p>
								</td>
								<td width="498">
								<p align="left">调用方法, func是方法名，args是参数，可以没有，或者有一个、多个参数.参数间用逗号隔开</p>
								</td>
							</tr>
						</tbody>
					</table>


这些操作符都是极其有用的，下面通过几个示例来演示它们的使用方法：

例9.4：几组操作符的示例

    ${pageScope.sampleValue + 12} <br>           //显示12
    ${(pageScope.sampleValue + 12)/3} <br>     //显示4.0
    ${(pageScope.sampleValue + 12) /3==4} <br>         //显示true
    ${(pageScope.sampleValue + 12) /3>=5} <br>         //显示false
    <input type="text" name="sample1" value="${pageScope.sampleValue + 10}"> //显示值为10的Text控件

可以看到，对于这些示例，程序设计者完全无需管理它们的类型转换，在表达式内部都已经处理了。有了EL表达式，在JSP页面的编程变得更灵活，也更容易。

### 标签库介绍
在JSTL1.1中有以下这些标签库是被支持的：Core标签库、XML processing标签库、I18N formatting标签库、Database access标签库、Functions标签库。
对应的标识符见表9.2所示：

表9.2 标签库的标识符

<table cellspacing="0" cellpadding="0" width="360" border="1">
						<tbody>
							<tr>
								<td width="107">
								<p align="left">标签库</p>
								</td>
								<td width="202">
								<p align="left">URI</p>
								</td>
								<td width="50">
								<p align="center">前缀</p>
								</td>
							</tr>
							<tr>
								<td width="107">
								<p align="left">Core</p>
								</td>
								<td width="202">
								<p align="left">http://java.sun.com/jsp/jstl/core</p>
								</td>
								<td width="50">
								<p align="left">c</p>
								</td>
							</tr>
							<tr>
								<td width="107">
								<p align="left">XML processing</p>
								</td>
								<td width="202">
								<p align="left">http://java.sun.com/jsp/jstl/xml</p>
								</td>
								<td width="50">
								<p align="left">x</p>
								</td>
							</tr>
							<tr>
								<td width="107">
								<p align="left">I18N formatting</p>
								</td>
								<td width="202">
								<p align="left">http://java.sun.com/jsp/jstl/fmt</p>
								</td>
								<td width="50">
								<p align="left">fmt</p>
								</td>
							</tr>
							<tr>
								<td width="107">
								<p align="left">Database access</p>
								</td>
								<td width="202">
								<p align="left">http://java.sun.com/jsp/jstl/sql</p>
								</td>
								<td width="50">
								<p align="left">sql</p>
								</td>
							</tr>
							<tr>
								<td width="107">
								<p align="left">Functions</p>
								</td>
								<td width="202">
								<p align="left">http://java.sun.com/jsp/jstl/functions</p>
								</td>
								<td width="50">
								<p align="left">fn</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看例9.5，简单使用标签库的示例。

例9.5：简单JSTL标签库示例

    <%@ page contentType="text/html; charset=UTF-8"%>
    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
    <html>
     <body>
        <c:forEach var="i" begin="1" end="10" step="1"> ${i} <br /></c:forEach>
     </body>
    </html>

在该示例的JSP页面中声明了将使用Core标签库，它的URI为“http://java.sun.com/jsp/jstl/core”，前缀为“c”。之后，页面中<c:forEach>标签就是使用了JSTL的标签进行了工作。对于该标签的功能，这里暂时不作具体讲解，只是让读者能够有个简单的概念，了解怎样定义和使用标签库。

### 标签库
Core标签库，又被称为核心标签库，该标签库的工作是对于JSP页面一般处理的封装。在该标签库中的标签一共有14个，被分为了四类，分别是：

- 多用途核心标签：<c:out>、<c:set>、<c:remove>、<c:catch>。
- 条件控制标签：<c:if>、<c:choose>、<c:when>、<c:otherwise>。
- 循环控制标签：<c:forEach>、<c:forTokens>。
- URL相关标签：<c:import>、<c:url>、<c:redirect>、<c:param>。

以下是各个标签的用途和属性以及简单示例。

### 用于显示的` <c:out>` 标签
`<c:out>`标签是一个最常用的标签，用于在JSP中显示数据。它的属性和描述如表9.3所示：

表9.3　`<c:out>`标签属性和说明

<table cellspacing="0" cellpadding="0" width="535" border="1">
						<tbody>
							<tr>
								<td width="79">
								<p align="left">属性</p>
								</td>
								<td width="456">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="79">
								<p align="left">value</p>
								</td>
								<td width="456">
								<p align="left">输出到页面的数据，可以是EL表达式或常量（必须）</p>
								</td>
							</tr>
							<tr>
								<td width="79">
								<p align="left">default</p>
								</td>
								<td width="456">
								<p align="left">当value为null时显示的数据（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="79">
								<p align="left">escapeXml</p>
								</td>
								<td width="456">
								<p align="left">当设置为true时会主动更换特殊字符，比如&ldquo;&amp;lt;,&amp;gt;,&amp;amp;&rdquo;（可选，默认为true）</p>
								</td>
							</tr>
						</tbody>
					</table>


在JSTL1.0的时候，在页面显示数据必须使用<c:out>来进行。然而，在JSTL1.1中，由于JSP2.0规范已经默认支持了EL表达式，因此可以直接在JSP页面使用表达式。下面看一个示例。

    <c:out value="${sessionScope.anyValue}" default="no value" escapeXml="false"/>

该示例将从Session查找名为“anyValue”的参数，并显示在页面，若没有找到则显示“no value”。

### `<c: >` 标签

#### 用于赋值的<c:set>标签
`<c:set>`标签用于为变量或JavaBean中的变量属性赋值的工作。它的属性和描述如表9.4所示：

表9.4　`<c:set>`标签属性和说明

<table cellpadding="0" width="393" border="1">
						<tbody>
							<tr>
								<td width="72">
								<p align="left">属性</p>
								</td>
								<td width="311">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">value</p>
								</td>
								<td>
								<p align="left">值的信息，可以是EL表达式或常量</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">target</p>
								</td>
								<td>
								<p align="left">被赋值的JavaBean实例的名称，若存在该属性则必须<br />
								存在property属性（可选）</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">property</p>
								</td>
								<td>
								<p align="left">JavaBean实例的变量属性名称（可选）</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">var</p>
								</td>
								<td>
								<p align="left">被赋值的变量名（可选）</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">scope</p>
								</td>
								<td>
								<p align="left">变量的作用范围，若没有指定，默认为page（可选）</p>
								</td>
							</tr>
						</tbody>
					</table>

 当不存在value的属性时，将以包含在标签内的实体数据作为赋值的内容。下面看一个示例：

    <c:set value="this is andy" var="oneString"/>
    ${oneString} <br>

该示例将为名为“oneString”的变量赋值为“this is andy”，其作用范围为page。

#### 9.3.3 用于删除的 `<c:remove> `标签
`<c:remove>`标签用于删除存在于scope中的变量。它的属性和描述如表9.5所示：

表9.5　`<c:remove>`标签属性和说明

<table cellspacing="0" cellpadding="0" width="365" border="1">
						<tbody>
							<tr>
								<td width="53">
								<p align="left">属性</p>
								</td>
								<td width="312">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="53">
								<p align="left">var</p>
								</td>
								<td width="312">
								<p align="left">需要被删除的变量名</p>
								</td>
							</tr>
							<tr>
								<td width="53">
								<p align="left">scope</p>
								</td>
								<td width="312">
								<p align="left">变量的作用范围，若没有指定，默认为全部查找（可选）</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例：

    <c:remove var="sampleValue" scope="session"/>
    ${sessionScope.sampleValue} <br>

该示例将存在于Session中名为“sampleValue”的变量删除。下一句EL表达式显示该变量时，该变量已经不存在了。

#### 9.3.4 用于异常捕获的 `<c:catch>` 标签
`<c:catch>`标签允许在JSP页面中捕捉异常。它包含一个var属性，是一个描述异常的变量，改变量可选。若没有var属性的定义，那么仅仅捕捉异常而不做任何事情，若定义了var属性，则可以利用var所定义的异常变量进行判断转发到其他页面或提示报错信息。看一个示例。

    <c:catch var="err"> ${param.sampleSingleValue[9] == 3}</c:catch>
    ${err}

当“${param.sampleSingleValue[9] == 3}”表达式有异常时，可以从var属性“err”得到异常的内容，通常判断“err”是否为null来决定错误信息的提示。

#### 9.3.5 用于判断的 `<c:if>` 标签
`<c:if>`标签用于简单的条件语句。它的属性和描述如表9.6所示：

表9.6　`<c:if>`标签属性和说明

<table cellspacing="0" cellpadding="0" width="454" border="1">
						<tbody>
							<tr>
								<td width="60">
								<p align="left">属性</p>
								</td>
								<td width="394">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="60">
								<p align="left">test</p>
								</td>
								<td width="394">
								<p align="left">需要判断的条件</p>
								</td>
							</tr>
							<tr>
								<td width="60">
								<p align="left">var</p>
								</td>
								<td width="394">
								<p align="left">保存判断结果true或false的变量名，该变量可供之后的工作使用（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="60">
								<p align="left">scope</p>
								</td>
								<td width="394">
								<p align="left">变量的作用范围，若没有指定，默认为保存于page范围中的变量（可选）</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例：

    <c:if test="${paramValues.sampleValue[2] == 12}" var="visits">It is 12</c:if><br>
    ${visits} <br>

该示例将判断request请求提交的传入控件数组参数中，下标为“2”的控件内容是否为“12”，若为12则显示“It is 12”。判断结果被保存在page范围中的“visits”变量中。

#### 9.3.6 用于复杂判断的 `<c:choose> 、 <c:when> 、 <c:otherwise>` 标签
这三个标签用于实现复杂条件判断语句，类似“if,elseif”的条件语句。

`<c:choose>`标签没有属性，可以被认为是父标签，`<c:when>、<c:otherwise>`将作为其子标签来使用。
`<c:when>`标签等价于“if”语句，它包含一个test属性，该属性表示需要判断的条件。
`<c:otherwise>`标签没有属性，它等价于“else”语句。

下面看一个复杂条件语句的示例。

    <c:choose>
             <c:when test="${paramValues.sampleValue[2] == 11}">
                       not 12 not 13,it is 11
             </c:when>
             <c:when test="${paramValues.sampleValue[2] == 12}">
                       not 11 not 13,it is 12
             </c:when>
             <c:when test="${paramValues.sampleValue[2] == 13}">
                       not 11 not 12,it is 13
             </c:when>
             <c:otherwise>
                       not 11 、12、13
             </c:otherwise>
    </c:choose>

该示例将判断request请求提交的传入控件数组参数中，下标为“2”控件内容是否为“11”或“12”或“13”，并根据判断结果显示各自的语句，若都不是则显示“not 11 、12、13”。

#### 9.3.7 用于循环的 `<c:forEach>` 标签
`<c:forEach>`为循环控制标签。它的属性和描述如表9.7所示：

表9.7　`<c:forEach>`标签属性和说明

<table cellspacing="0" cellpadding="0" width="511" border="1">
						<tbody>
							<tr>
								<td width="59">
								<p align="left">属性</p>
								</td>
								<td width="452">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="59">
								<p align="left">items</p>
								</td>
								<td width="452">
								<p align="left">进行循环的集合（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="59">
								<p align="left">begin</p>
								</td>
								<td width="452">
								<p align="left">开始条件（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="59">
								<p align="left">end</p>
								</td>
								<td width="452">
								<p align="left">结束条件（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="59">
								<p align="left">step</p>
								</td>
								<td width="452">
								<p align="left">循环的步长，默认为1（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="59">
								<p align="left">var</p>
								</td>
								<td width="452">
								<p align="left">做循环的对象变量名，若存在items属性，则表示循环集合中对象的变量名（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="59">
								<p align="left">varStatus</p>
								</td>
								<td width="452">
								<p align="left">显示循环状态的变量（可选）</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个集合循环的示例。

<%ArrayList arrayList = new ArrayList();
                   arrayList.add("aa");
                   arrayList.add("bb");
                   arrayList.add("cc");
%>
<%request.getSession().setAttribute("arrayList", arrayList);%>
<c:forEach items="${sessionScope.arrayList}" var="arrayListI">
         ${arrayListI}
</c:forEach>

该示例将保存在Session中的名为“arrayList”的ArrayList类型集合参数中的对象依次读取出来，items属性指向了ArrayList类型集合参数，var属性定义了一个新的变量来接收集合中的对象。最后直接通过EL表达式显示在页面上。下面看一个简单循环的示例。

    <c:forEach var="i" begin="1" end="10" step="1">
          ${i}<br />
    </c:forEach>

该示例从“1”循环到“10”，并将循环中变量“i”显示在页面上。

#### 9.3.8 用于分隔字符的 `<c:forTokens>` 标签
`<c:forTokens>`标签可以根据某个分隔符分隔指定字符串，相当于java.util.StringTokenizer类。它的属性和描述如表9.8所示：

表9.8　`<c:forTokens>`标签属性和说明

<table cellspacing="0" cellpadding="0" width="268" border="1">
						<tbody>
							<tr>
								<td width="76">
								<p align="left">属性</p>
								</td>
								<td width="192">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="76">
								<p align="left">items</p>
								</td>
								<td width="192">
								<p align="left">进行分隔的EL表达式或常量</p>
								</td>
							</tr>
							<tr>
								<td width="76">
								<p align="left">delims</p>
								</td>
								<td width="192">
								<p align="left">分隔符</p>
								</td>
							</tr>
							<tr>
								<td width="76">
								<p align="left">begin</p>
								</td>
								<td width="192">
								<p align="left">开始条件（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="76">
								<p align="left">end</p>
								</td>
								<td width="192">
								<p align="left">结束条件（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="76">
								<p align="left">step</p>
								</td>
								<td width="192">
								<p align="left">循环的步长，默认为1（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="76">
								<p align="left">var</p>
								</td>
								<td width="192">
								<p align="left">做循环的对象变量名（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="76">
								<p align="left">varStatus</p>
								</td>
								<td width="192">
								<p align="left">显示循环状态的变量（可选）</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例。

    <c:forTokens items="aa,bb,cc,dd" begin="0" end="2" step="2" delims="," var="aValue">
             ${aValue}
    </c:forTokens>

需要分隔的字符串为“aa,bb,cc,dd”，分隔符为“,”。begin属性指定从第一个“,”开始分隔，end属性指定分隔到第三个“,”，并将做循环的变量名指定为“aValue”。由于步长为“2”，使用EL表达式${aValue}只能显示“aa

### `<x: >` 标签



#### 9.3.9 用于包含页面的`<c:import>`
`<c:import>`标签允许包含另一个JSP页面到本页面来。它的属性和描述如表9.9所示：

表9.9　`<c:import>`标签属性和说明
<table cellpadding="0" width="360" border="1">
						<tbody>
							<tr>
								<td width="73">
								<p align="left">属性</p>
								</td>
								<td width="271">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">rl</p>
								</td>
								<td>
								<p align="left">需要导入页面的URL</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">context</p>
								</td>
								<td>
								<p align="left">Web Context该属性用于在不同的Context下导入页面，当出现context属性时，必须以&ldquo;/&rdquo;开头，此时也需要url属性以&ldquo;/&rdquo;开头（可选）</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">charEncoding</p>
								</td>
								<td>
								<p align="left">导入页面的字符集（可选）</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">var</p>
								</td>
								<td>
								<p align="left">可以定义导入文本的变量名（可选）</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">scope</p>
								</td>
								<td>
								<p align="left">导入文本的变量名作用范围（可选）</p>
								</td>
							</tr>
							<tr>
								<td>
								<p align="left">varReader</p>
								</td>
								<td>
								<p align="left">接受文本的java.io.Reader类变量名（可选）</p>
								</td>
							</tr>
						</tbody>
					</table>


 下面看一个示例。

    <c:import url="/MyHtml.html" var="thisPage" />
    <c:import url="/MyHtml.html" context=”/sample2” var="thisPage"/>
    <c:import url="www.sample.com/MyHtml.html" var="thisPage"/>

该示例演示了三种不同的导入方法，第一种是在同一 Context 下的导入，第二种是在不同的 Context 下导入，第三种是导入任意一个 URL 。

#### 9.3.10 用于得到URL地址的`<c:url>`标签
`<c:url>` 标签用于得到一个 URL 地址。它的属性和描述如表 9.10 所示：

表9.10 `<c:url>`标签属性和说明

<table cellspacing="0" cellpadding="0" width="459" border="1">
						<tbody>
							<tr>
								<td width="81">
								<p align="left">属性</p>
								</td>
								<td width="378">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="81">
								<p align="left">value</p>
								</td>
								<td width="378">
								<p align="left">页面的URL地址</p>
								</td>
							</tr>
							<tr>
								<td width="81">
								<p align="left">context</p>
								</td>
								<td width="378">
								<p align="left">Web Context该属性用于得到不同Context下的URL地址，当出现context属性时，必须以&ldquo;/&rdquo;开头，此时也需要url属性以&ldquo;/&rdquo;开头（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="81">
								<p align="left">charEncoding</p>
								</td>
								<td width="378">
								<p align="left">URL的字符集（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="81">
								<p align="left">var</p>
								</td>
								<td width="378">
								<p align="left">存储URL的变量名（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="81">
								<p align="left">scope</p>
								</td>
								<td width="378">
								<p align="left">变量名作用范围（可选）</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例：

    <c:url value="/MyHtml.html" var="urlPage" />
    <a href="${urlPage}">link</a>

得到了一个 URL 后，以 EL 表达式放入` <a>` 标签的 href 属性，达到链接的目的。

#### 9.3.11 用于页面重定向的`<c:redirect>`标签
`<c:redirect>` 用于页面的重定向，该标签的作用相当于 `response.setRedirect` 方法的工作。它包含 url 和 context 两个属性，属性含义和 `<c:url>` 标签相同。下面看一个示例。

    <c:redirect url="/MyHtml.html"/>

该示例若出现在 JSP 中，则将重定向到当前 Web Context 下的“ MyHtml.html ”页面，一般会与 `<c:if>` 等标签一起使用。

#### 9.3.12 用于包含传递参数的`<c:param>`标签
 `<c:param>` 用来为包含或重定向的页面传递参数。它的属性和描述如表 9.11 所示：

表9.11　`<c:param>`标签属性和说明

<table cellspacing="0" cellpadding="0" width="190" border="1">
						<tbody>
							<tr>
								<td width="58">
								<p align="left">属性</p>
								</td>
								<td width="132">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="58">
								<p align="left">name</p>
								</td>
								<td width="132">
								<p align="left">传递的参数名</p>
								</td>
							</tr>
							<tr>
								<td width="58">
								<p align="left">value</p>
								</td>
								<td width="132">
								<p align="left">传递的参数值（可选）</p>
								</td>
							</tr>
						</tbody>
					</table>


下面是一个示例：

    <c:redirect url="/MyHtml.jsp">
    <c:param name="userName" value=”RW” />
    </c:redirect>

该示例将为重定向的“ MyHtml.jsp ”传递指定参数“ userName=’RW’ ”。

### 9.4 JSTL XML processing标签库
在企业级应用越来越依赖 XML 的今天， XML 格式的数据被作为信息交换的优先选择。 XML processing 标签库为程序设计者提供了基本的对 XML 格式文件的操作。在该标签库中的标签一共有 10 个，被分为了三类，分别是：

- XML核心标签：`<x:parse>、<x:out>、<x:set>`。
- XML流控制标签：`<x:if>、<x:choose>、<x:when>、<x:otherwise>、<x:forEach>`。
- XML转换标签：`<x:transform>、<x:param>`。

由于该组标签库专注于对某一特定领域的实现，因此本书将只选择其中常见的一些标签和属性进行介绍。

#### 9.4.1 用于解析XML文件的`<x:parse>`标签
`<x:parse>` 标签是该组标签库的核心，从其标签名就可以知道，它是作为解析 XML 文件而存在的。它的属性和描述如表 9.12 所示：

表9.12　`<x:parse>`标签属性和说明

<table cellspacing="0" cellpadding="0" width="537" border="1">
						<tbody>
							<tr>
								<td width="67">
								<p align="left">属性</p>
								</td>
								<td width="470">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="67">
								<p align="left">doc</p>
								</td>
								<td width="470">
								<p align="left">源XML的内容，该属性的内容应该为String类型或者java.io.Reader的实例，可以用xml属性来替代，但是不被推荐</p>
								</td>
							</tr>
							<tr>
								<td width="67">
								<p align="left">var</p>
								</td>
								<td width="470">
								<p align="left">将解析后的XML保存在该属性所指定的变量中，之后XML processing标签库中的其他标签若要取XML中的内容就可以从该变量中得到（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="67">
								<p align="left">scope</p>
								</td>
								<td width="470">
								<p align="left">变量的作用范围（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="67">
								<p align="left">varDom</p>
								</td>
								<td width="470">
								<p align="left">指定保存的变量为org.w3c.dom.Document接口类型（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="67">
								<p align="left">scopeDom</p>
								</td>
								<td width="470">
								<p align="left">org.w3c.dom.Document的接口类型变量作用范围（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="67">
								<p align="left">systemId</p>
								</td>
								<td width="470">
								<p align="left">定义一个URI，该URI将被使用到XML文件中以接入其他资源文件（可选）</p>
								</td>
							</tr>
							<tr>
								<td width="67">
								<p align="left">filter</p>
								</td>
								<td width="470">
								<p align="left">该属性必须为org.xml.sax.XMLFilter类的一个实例，可以使用EL表达式传入，将对XML文件做过滤得到自身需要的部分（可选）</p>
								</td>
							</tr>
						</tbody>
					</table>


其中， var 、 scope 和 varDom 、 scopeDom 不应该同时出现，而应该被视为两个版本来使用，二者的变量都可以被 XML processing 标签库的其他标签来使用。

`<x:parse>`标签单独使用的情况很少，一般会结合 XML processing 标签库中的其他标签来一起工作。下面看一个示例。

首先给出一个简单的 XML 文件，将对该 XML 文件做解析，该 XML 文件名为 SampleXml.xml 。

    <?xml version="1.0" encoding="UTF-8"?>
    <xml-body>
             <name>RW</name>
             <passWord>123456</passWord>
             <age>28</age>
             <books>
                       <book>book1</book>
                       <book>book2</book>
                       <book>book3</book>
             </books>
    </xml-body>

标签库的工作：

    <c:import var="xmlFile" url="http://localhost:8080/booksamplejstl/SampleXml.xml"/>
    <x:parse var="xmlFileValue" doc="${xmlFile}"/>

### `<fmt: >` 标签

看到I18N就应该想到知识“国际化”，I18N formatting标签库就是用于在JSP页面中做国际化的动作。在该标签库中的标签一共有12个，被分为了两类，分别是：

- 国际化核心标签：`<fmt:setLocale>、<fmt:bundle>、<fmt:setBundle>、<fmt:message>、<fmt:param>、<fmt:requestEncoding>`。
- 格式化标签：`<fmt:timeZone>、<fmt:setTimeZone>、<fmt:formatNumber>、<fmt:parseNumber>、<fmt:formatDate>、<fmt:parseDate>`。

下面只选择其中常见的一些标签和属性进行介绍。

#### 9.5.1 用于设置本地化环境的`<fmt:setLocale>`标签
    `<fmt:setLocale>`标签用于设置Locale环境。它的属性和描述如表9.17所示：

表9.17　`<fmt:setLocale>`标签属性和说明

<table cellspacing="0" cellpadding="0" border="1">
						<tbody>
							<tr>
								<td width="49">
								<p align="left">属性</p>
								</td>
								<td width="336">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="49">
								<p align="left">value</p>
								</td>
								<td width="336">
								<p align="left">Locale环境的指定，可以是java.util.Locale或String类型的实例</p>
								</td>
							</tr>
							<tr>
								<td width="49">
								<p align="left">scope</p>
								</td>
								<td width="336">
								<p align="left">Locale环境变量的作用范围（可选）</p>
								</td>
							</tr>
						</tbody>
					</table>

下面看一个示例：

    <fmt:setLocale value="zh_TW"/>

表示设置本地环境为繁体中文。

#### 9.5.2 用于资源文件绑定的`<fmt:bundle>、<fmt:setBundle>`标签

这两组标签用于资源配置文件的绑定，唯一不同的是`<fmt:bundle>`标签将资源配置文件绑定于它标签体中的显示，`<fmt:setBundle>`标签则允许将资源配置文件保存为一个变量，在之后的工作可以根据该变量来进行。

根据Locale环境的不同将查找不同后缀的资源配置文件，这点在国际化的任何技术上都是一致的，通常来说，这两种标签单独使用是没有意义的，它们都会与I18N formatting标签库中的其他标签配合使用。它们的属性和描述如表9.18所示：


表9.18　`<fmt:bundle>、<fmt:setBundle>`标签属性和说明

<table cellspacing="0" cellpadding="0" border="1">
						<tbody>
							<tr>
								<td width="69">
								<p align="left">属性</p>
								</td>
								<td width="421">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="69">
								<p align="left">basename</p>
								</td>
								<td width="421">
								<p align="left">资源配置文件的指定，只需要指定文件名而无须扩展名，二组标签共有的属性</p>
								</td>
							</tr>
							<tr>
								<td width="69">
								<p align="left">var</p>
								</td>
								<td width="421">
								<p align="left">&lt;fmt:setBundle&gt;独有的属性，用于保存资源配置文件为一个变量</p>
								</td>
							</tr>
							<tr>
								<td width="69">
								<p align="left">scope</p>
								</td>
								<td width="421">
								<p align="left">变量的作用范围</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例

    <fmt:setLocale value="zh_CN"/>
    <fmt:setBundle basename="applicationMessage" var="applicationBundle"/>

该示例将会查找一个名为applicationMessage_zh_CN.properties的资源配置文件，来作为显示的Resource绑定。

#### 9.5.3  用于显示资源配置文件信息的`<fmt:message>`标签
用于信息显示的标签，将显示资源配置文件中定义的信息。它的属性和描述如表9.19所示：

表9.19　`<fmt:message>`标签属性和说明

<table cellspacing="0" cellpadding="0" border="1">
						<tbody>
							<tr>
								<td width="55">
								<p align="left">属性</p>
								</td>
								<td width="513">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="55">
								<p align="left">key</p>
								</td>
								<td width="513">
								<p align="left">资源配置文件的&ldquo;键&rdquo;指定</p>
								</td>
							</tr>
							<tr>
								<td width="55">
								<p align="left">bundle</p>
								</td>
								<td width="513">
								<p align="left">若使用&lt;fmt:setBundle&gt;保存了资源配置文件，该属性就可以从保存的资源配置文件中进行查找</p>
								</td>
							</tr>
							<tr>
								<td width="55">
								<p align="left">var</p>
								</td>
								<td width="513">
								<p align="left">将显示信息保存为一个变量</p>
								</td>
							</tr>
							<tr>
								<td width="55">
								<p align="left">scope</p>
								</td>
								<td width="513">
								<p align="left">变量的作用范围</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例：

    <fmt:setBundle basename="applicationMessage" var="applicationBundle"/>
    <fmt:bundle basename="applicationAllMessage">
             <fmt:message key="userName" />
             <p>
             <fmt:message key="passWord" bundle="${applicationBundle}" />
    </fmt:bundle>

该示例使用了两种资源配置文件的绑定的做法，“ applicationMessage ”资源配置文件利用`<fmt:setBundle>`标签被赋于了变量“ applicationBundle ”，而作为`<fmt:bundle>`标签定义的“ applicationAllMessage ”资源配置文件作用于其标签体内的显示。

第一个`<fmt:message>`标签将使用“ applicationAllMessage ”资源配置文件中“键”为“ userName ”的信息显示。
第二个`<fmt:message>`标签虽然被定义在`<fmt:bundle>`标签体内，但是它使用了bundle属性，因此将指定之前由`<fmt:setBundle>`标签保存的“ applicationMessage ”资源配置文件，该“键”为“ passWord ”的信息显示。

#### 9.5.4  用于参数传递的`<fmt:param>`标签
`<fmt:param>`标签应该位于`<fmt:message>`标签内，将为该消息标签提供参数值。它只有一个属性value。
`<fmt:param>`标签有两种使用版本，一种是直接将参数值写在value属性中，另一种是将参数值写在标签体内。

#### 9.5.6  用于为请求设置字符编码的`<fmt:requestEncoding>`标签
`<fmt:requestEncoding>`标签用于为请求设置字符编码。它只有一个属性value，在该属性中可以定义字符编码。

#### 9.5.7  用于设定时区的`<fmt:timeZone>、<fmt:setTimeZone>`标签
这两组标签都用于设定一个时区。唯一不同的是`<fmt:timeZone>`标签将使得在其标签体内的工作可以使用该时区设置，`<fmt:setBundle>`标签则允许将时区设置保存为一个变量，在之后的工作可以根据该变量来进行。它们的属性和描述如表9.20所示：

表9.20　`<fmt:timeZone>、<fmt:setTimeZone>`标签属性和说明

<table cellspacing="0" cellpadding="0" border="1">
						<tbody>
							<tr>
								<td width="53">
								<p align="left">属性</p>
								</td>
								<td width="312">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="53">
								<p align="left">value</p>
								</td>
								<td width="312">
								<p align="left">时区的设置</p>
								</td>
							</tr>
							<tr>
								<td width="53">
								<p align="left">var</p>
								</td>
								<td width="312">
								<p align="left">&lt;fmt:setTimeZone&gt;独有的属性，用于保存时区为一个变量</p>
								</td>
							</tr>
							<tr>
								<td width="53">
								<p align="left">scope</p>
								</td>
								<td width="312">
								<p align="left">变量的作用范围</p>
								</td>
							</tr>
						</tbody>
					</table>


#### 9.5.8  用于格式化数字的`<fmt:formatNumber>`标签
`<fmt: formatNumber >` 标签用于格式化数字。它的属性和描述如表9.21所示：

表9.21　`<fmt:formatNumber>`标签属性和说明

<table cellspacing="0" cellpadding="0" border="1">
						<tbody>
							<tr>
								<td width="104">
								<p align="left">属性</p>
								</td>
								<td width="384">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="104">
								<p align="left">value</p>
								</td>
								<td width="384">
								<p align="left">格式化的数字，该数值可以是String类型或java.lang.Number类型的实例</p>
								</td>
							</tr>
							<tr>
								<td width="104">
								<p align="left">type</p>
								</td>
								<td width="384">
								<p align="left">格式化的类型</p>
								</td>
							</tr>
							<tr>
								<td width="104">
								<p align="left">pattern</p>
								</td>
								<td width="384">
								<p align="left">格式化模式</p>
								</td>
							</tr>
							<tr>
								<td width="104">
								<p align="left">var</p>
								</td>
								<td width="384">
								<p align="left">结果保存变量</p>
								</td>
							</tr>
							<tr>
								<td width="104">
								<p align="left">scope</p>
								</td>
								<td width="384">
								<p align="left">变量的作用范围</p>
								</td>
							</tr>
							<tr>
								<td width="104">
								<p align="left">maxIntegerDigits</p>
								</td>
								<td width="384">
								<p align="left">指定格式化结果的最大值</p>
								</td>
							</tr>
							<tr>
								<td width="104">
								<p align="left">minIntegerDigits</p>
								</td>
								<td width="384">
								<p align="left">指定格式化结果的最小值</p>
								</td>
							</tr>
							<tr>
								<td width="104">
								<p align="left">maxFractionDigits</p>
								</td>
								<td width="384">
								<p align="left">指定格式化结果的最大值，带小数</p>
								</td>
							</tr>
							<tr>
								<td width="104">
								<p align="left">minFractionDigits</p>
								</td>
								<td width="384">
								<p align="left">指定格式化结果的最小值，带小数</p>
								</td>
							</tr>
						</tbody>
					</table>


`<fmt:formatNumber>`标签实际是对应java.util.NumberFormat类，type属性的可能值包括currency（货币）、number（数字）和percent（百分比）。

下面看一个示例。

    <fmt:formatNumber value="1000.888" type="currency" var="money"/>

该结果将被保存在“ money ”变量中，将根据Locale环境显示当地的货币格式。

#### 9.5.9  用于解析数字的`<fmt:parseNumber>`标签
`<fmt:parseNumber>`标签用于解析一个数字，并将结果作为java.lang.Number类的实例返回。`<fmt:parseNumber>`标签看起来和`<fmt:formatNumber>`标签的作用正好相反。它的属性和描述如表9.22所示：

表9.22　`<fmt:parseNumber>`标签属性和说明

<table cellspacing="0" cellpadding="0" border="1">
						<tbody>
							<tr>
								<td width="72">
								<p align="left">属性</p>
								</td>
								<td width="463">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">value</p>
								</td>
								<td width="463">
								<p align="left">将被解析的字符串</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">type</p>
								</td>
								<td width="463">
								<p align="left">解析格式化的类型</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">pattern</p>
								</td>
								<td width="463">
								<p align="left">解析格式化模式</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">var</p>
								</td>
								<td width="463">
								<p align="left">结果保存变量，类型为java.lang.Number</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">scope</p>
								</td>
								<td width="463">
								<p align="left">变量的作用范围</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">parseLocale</p>
								</td>
								<td width="463">
								<p align="left">以本地化的形式来解析字符串，该属性的内容应为String或java.util.Locale类型的实例</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例。

    <fmt:parseNumber value="15%" type="percent" var="num"/>

解析之后的结果为“ 0.15 ”。

#### 9.5.10  用于格式化日期的`<fmt:formatDate>`标签
`<fmt:formatDate>`标签用于格式化日期。它的属性和描述如表9.23所示：

表9.23　`<fmt:formatDate>`标签属性和说明

<table cellspacing="0" cellpadding="0" border="1">
						<tbody>
							<tr>
								<td width="105">
								<p align="left">属性</p>
								</td>
								<td width="343">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="105">
								<p align="left">value</p>
								</td>
								<td width="343">
								<p align="left">格式化的日期，该属性的内容应该是java.util.Date类型的实例</p>
								</td>
							</tr>
							<tr>
								<td width="105">
								<p align="left">type</p>
								</td>
								<td width="343">
								<p align="left">格式化的类型</p>
								</td>
							</tr>
							<tr>
								<td width="105">
								<p align="left">pattern</p>
								</td>
								<td width="343">
								<p align="left">格式化模式</p>
								</td>
							</tr>
							<tr>
								<td width="105">
								<p align="left">var</p>
								</td>
								<td width="343">
								<p align="left">结果保存变量</p>
								</td>
							</tr>
							<tr>
								<td width="105">
								<p align="left">scope</p>
								</td>
								<td width="343">
								<p align="left">变量的作用范围</p>
								</td>
							</tr>
							<tr>
								<td width="105">
								<p align="left">timeZone</p>
								</td>
								<td width="343">
								<p align="left">指定格式化日期的时区</p>
								</td>
							</tr>
						</tbody>
					</table>


`<fmt:formatDate>`标签与`<fmt:timeZone>、<fmt:setTimeZone>`两组标签的关系密切。若没有指定 timeZone属性，也可以通过`<fmt:timeZone>、<fmt:setTimeZone>`两组标签设定的时区来格式化最后的结果。

#### 9.5.11  用于解析日期的`<fmt:parseDate>`标签
`<fmt:parseDate>`标签用于解析一个日期，并将结果作为java.lang.Date类型的实例返回。`<fmt:parseDate>`标签看起来和`<fmt:formatDate>`标签的作用正好相反。它的属性和描述如表9.24所示：

表9.24　`<fmt:parseDate>`标签属性和说明

<table cellspacing="0" cellpadding="0" border="1">
						<tbody>
							<tr>
								<td width="72">
								<p align="left">属性</p>
								</td>
								<td width="451">
								<p align="left">描述</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">value</p>
								</td>
								<td width="451">
								<p align="left">将被解析的字符串</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">type</p>
								</td>
								<td width="451">
								<p align="left">解析格式化的类型</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">pattern</p>
								</td>
								<td width="451">
								<p align="left">解析格式化模式</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">var</p>
								</td>
								<td width="451">
								<p align="left">结果保存变量，类型为java.lang.Date</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">scope</p>
								</td>
								<td width="451">
								<p align="left">变量的作用范围</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">parseLocale</p>
								</td>
								<td width="451">
								<p align="left">以本地化的形式来解析字符串，该属性的内容为String或java.util.Locale类型的实例</p>
								</td>
							</tr>
							<tr>
								<td width="72">
								<p align="left">timeZone</p>
								</td>
								<td width="451">
								<p align="left">指定解析格式化日期的时区</p>
								</td>
							</tr>
						</tbody>
					</table>


`<fmt:parseNumber>`和`<fmt:parseDate>`两组标签都实现解析字符串为一个具体对象实例的工作，因此，这两组解析标签对var属性的字符串参数要求非常严格。就JSP页面的表示层前段来说，处理这种解析本不属于份内之事，因此`<fmt:parseNumber>`和`<fmt:parseDate>`两组标签应该尽量少用，替代工作的地方应该在服务器端表示层的后段，比如在Servlet中。

 `<sql:>`标签

### 9.6  Database access 标签库

Database access标签库中的标签用来提供在JSP页面中可以与数据库进行交互的功能，虽然它的存在对于早期纯JSP开发的应用以及小型的开发有着意义重大的贡献，但是对于MVC模型来说，它却是违反规范的。因为与数据库交互的工作本身就属于业务逻辑层的工作，所以不应该在JSP页面中出现，而是应该在模型层中进行。

对于Database access标签库本书不作重点介绍，只给出几个简单示例让读者略微了解它们的功能。

Database access标签库有以下6组标签来进行工作：`<sql:setDataSource>、<sql:query>、<sql:update>、<sql:transaction>、<sql:setDataSource>、<sql:param>、<sql:dateParam>`。

#### 9.6.1 用于设置数据源的 `<sql:setDataSource>` 标签
`<sql:setDataSource>`标签用于设置数据源，下面看一个示例：

    <sql:setDataSource
             var="dataSrc"
             url="jdbc:postgresql://localhost:5432/myDB"
             driver="org.postgresql.Driver"
             user="admin"
             password="1111"/>

该示例定义一个数据源并保存在“ dataSrc ”变量内。

#### 9.6.2 用于查询的 `<sql:query>`标签

`<sql:query>`标签用于查询数据库，它标签体内可以是一句查询SQL。下面看一个示例：

    <sql:query var="queryResults" dataSource="${dataSrc}">
          select * from table1
    </sql:query>

该示例将返回查询的结果到变量“ queryResults ”中，保存的结果是javax.servlet.jsp.jstl.sql.Result类型的实例。要取得结果集中的数据可以使用`<c:forEach>`循环来进行。下面看一个示例。

    <c:forEach var="row" items="${queryResults.rows}">
          <tr>
                   <td>${row.userName}</td>
                    <td>${row.passWord}</td>
          </tr>
    </c:forEach>

“ rows ”是javax.servlet.jsp.jstl.sql.Result实例的变量属性之一，用来表示数据库表中的“列”集合，循环时，通过“ ${row.XXX} ”表达式可以取得每一列的数据，“ XXX ”是表中的列名。

#### 9.6.3 用于更新的 `<sql:update>` 标签
`<sql:update>`标签用于更新数据库，它的标签体内可以是一句更新的SQL语句。其使用和`<sql:query>`标签没有什么不同。

#### 9.6.4 用于事务处理的` <sql:transaction> `标签
`<sql:transaction>`标签用于数据库的事务处理，在该标签体内可以使用`<sql:update>`标签和`<sql:query>`标签，而`<sql:transaction>`标签的事务管理将作用于它们之上。
`<sql:transaction>`标签对于事务处理定义了read_committed、read_uncommitted、repeatable_read、serializable4个隔离级别。

#### 9.6.5 用于事务处理的` <sql:param> 、 <sql:dateParam>` 标签
这两个标签用于向SQL语句提供参数，就好像程序中预处理SQL的“ ? ”一样。`<sql:param>`标签传递除java.util.Date类型以外的所有相融参数，`<sql:dateParam>`标签则指定必须传递java.util.Date类型的参数。

### `<fn:>` 标签

### 9.7 Functions 标签库
       称呼Functions标签库为标签库，倒不如称呼其为函数库来得更容易理解些。因为Functions标签库并没有提供传统的标签来为JSP页面的工作服务，而是被用于EL表达式语句中。在JSP2.0规范下出现的Functions标签库为EL表达式语句提供了许多更为有用的功能。Functions标签库分为两大类，共16个函数。

       长度函数：fn:length
       字符串处理函数：fn:contains、fn:containsIgnoreCase、fn:endsWith、fn:escapeXml、fn:indexOf、fn:join、fn:replace、fn:split、fn:startsWith、fn:substring、fn:substringAfter、fn:substringBefore、fn:toLowerCase、fn:toUpperCase、fn:trim

以下是各个函数的用途和属性以及简单示例。

#### 9.7.1 长度函数 fn:length 函数
       长度函数fn:length的出现有重要的意义。在JSTL1.0中，有一个功能被忽略了，那就是对集合的长度取值。虽然java.util.Collection接口定义了size方法，但是该方法不是一个标准的JavaBean属性方法（没有get,set方法），因此，无法通过EL表达式“ ${collection.size} ”来轻松取得。

fn:length函数正是为了解决这个问题而被设计出来的。它的参数为input，将计算通过该属性传入的对象长度。该对象应该为集合类型或String类型。其返回结果是一个int类型的值。下面看一个示例。

    <%ArrayList arrayList1 = new ArrayList();
                                arrayList1.add("aa");
                                arrayList1.add("bb");
                                arrayList1.add("cc");

    %>
    <%request.getSession().setAttribute("arrayList1", arrayList1);%>
    ${fn:length(sessionScope.arrayList1)}

假设一个ArrayList类型的实例“ arrayList1 ”，并为其添加三个字符串对象，使用fn:length函数后就可以取得返回结果为“ 3 ”。

#### 9.7.2 判断函数 fn:contains 函数
       fn:contains函数用来判断源字符串是否包含子字符串。它包括string和substring两个参数，它们都是String类型，分布表示源字符串和子字符串。其返回结果为一个boolean类型的值。下面看一个示例。

    ${fn:contains("ABC", "a")}<br>
    ${fn:contains("ABC", "A")}<br>

前者返回“ false ”，后者返回“ true ”。

#### 9.7.3 fn:containsIgnoreCase函数
      fn:containsIgnoreCase函数与fn:contains函数的功能差不多，唯一的区别是fn:containsIgnoreCase函数对于子字符串的包含比较将忽略大小写。它与fn:contains函数相同，包括string和substring两个参数，并返回一个boolean类型的值。下面看一个示例。

    ${fn:containsIgnoreCase("ABC", "a")}<br>
    ${fn:containsIgnoreCase("ABC", "A")}<br>

前者和后者都会返回“ true ”。


#### 9.7.4 词头判断函数 fn:startsWith 函数
       fn:startsWith函数用来判断源字符串是否符合一连串的特定词头。它除了包含一个string参数外，还包含一个subffx参数，表示词头字符串，同样是String类型。该函数返回一个boolean类型的值。下面看一个示例。

    ${fn:startsWith ("ABC", "ab")}<br>
    ${fn:startsWith ("ABC", "AB")}<br>

前者返回“ false ”，后者返回“ true ”。


#### 9.7.5 词尾判断函数 fn:endsWith 函数
       fn:endsWith函数用来判断源字符串是否符合一连串的特定词尾。它与fn:startsWith函数相同，包括string和subffx两个参数，并返回一个boolean类型的值。下面看一个示例。

    ${fn:endsWith("ABC", "bc")}<br>
    ${fn:endsWith("ABC", "BC")}<br>

前者返回“ false ”，后者返回“ true ”。


#### 9.7.6 字符实体转换函数 fn:escapeXml 函数
fn:escapeXml函数用于将所有特殊字符转化为字符实体码。它只包含一个string参数，返回一个String类型的值。


#### 9.7.8 字符匹配函数 fn:indexOf 函数
       fn:indexOf函数用于取得子字符串与源字符串匹配的开始位置，若子字符串与源字符串中的内容没有匹配成功将返回“ -1 ”。它包括string和substring两个参数，返回结果为int类型。下面看一个示例。

    ${fn:indexOf("ABCD","aBC")}<br>
    ${fn:indexOf("ABCD","BC")}<br>

前者由于没有匹配成功，所以返回-1，后者匹配成功将返回位置的下标，为1。

#### 9.7.9 分隔符函数 fn:join 函数
       fn:join函数允许为一个字符串数组中的每一个字符串加上分隔符，并连接起来。它的参数、返回结果和描述如表9.25所示：

表9.25　fn:join函数

<table cellspacing="0" cellpadding="0" width="350" border="1">
						<tbody>
							<tr>
								<td width="65">
								<p>参数</p>
								</td>
								<td width="216">
								<p>描述</p>
								</td>
							</tr>
							<tr>
								<td width="65">
								<p>array</p>
								</td>
								<td width="216">
								<p>字符串数组。其类型必须为String[]类型</p>
								</td>
							</tr>
							<tr>
								<td width="65">
								<p>separator</p>
								</td>
								<td width="216">
								<p>分隔符。其类型必须为String类型</p>
								</td>
							</tr>
							<tr>
								<td width="65">
								<p>返回结果</p>
								</td>
								<td width="216">
								<p>返回一个String类型的值</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例。

    <% String[] stringArray = {"a","b","c"}; %>
    <%request.getSession().setAttribute("stringArray", stringArray);%>
    ${fn:join(sessionScope.stringArray,";")}<br>

定义数组并放置到Session中，然后通过Session得到该字符串数组，使用fn:join函数并传入分隔符“ ; ”，得到的结果为“ a;b;c ”。


#### 9.7.10 替换函数 fn:replace 函数
       fn:replace函数允许为源字符串做替换的工作。它的参数、返回结果和描述如表9.26所示：

表9.26　fn:replace函数

<table cellspacing="0" cellpadding="0" width="353" border="1">
						<tbody>
							<tr>
								<td width="105">
								<p>参数</p>
								</td>
								<td width="249">
								<p>描述</p>
								</td>
							</tr>
							<tr>
								<td width="105">
								<p>inputString</p>
								</td>
								<td width="249">
								<p>源字符串。其类型必须为String类型</p>
								</td>
							</tr>
							<tr>
								<td width="105">
								<p>beforeSubstring</p>
								</td>
								<td width="249">
								<p>指定被替换字符串。其类型必须为String类型</p>
								</td>
							</tr>
							<tr>
								<td width="105">
								<p>afterSubstring</p>
								</td>
								<td width="249">
								<p>指定替换字符串。其类型必须为String类型</p>
								</td>
							</tr>
							<tr>
								<td width="105">
								<p>返回结果</p>
								</td>
								<td width="249">
								<p>返回一个String类型的值</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例。

${fn:replace("ABC","A","B")}<br>

将“ ABC ”字符串替换为“ BBC ”，在“ ABC ”字符串中用“ B ”替换了“ A ”。


#### 9.7.11 分隔符转换数组函数 fn:split 函数
fn:split函数用于将一组由分隔符分隔的字符串转换成字符串数组。它的参数、返回结果和描述如表9.27所示：

表9.27　fn:split函数

<table cellspacing="0" cellpadding="0" width="281" border="1">
						<tbody>
							<tr>
								<td width="65">
								<p>参数</p>
								</td>
								<td width="216">
								<p>描述</p>
								</td>
							</tr>
							<tr>
								<td width="65">
								<p>string</p>
								</td>
								<td width="216">
								<p>源字符串。其类型必须为String类型</p>
								</td>
							</tr>
							<tr>
								<td width="65">
								<p>delimiters</p>
								</td>
								<td width="216">
								<p>指定分隔符。其类型必须为String类型</p>
								</td>
							</tr>
							<tr>
								<td width="65">
								<p>返回结果</p>
								</td>
								<td width="216">
								<p>返回一个String[]类型的值</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例。

${fn:split("A,B,C",",")}<br>

将“ A,B,C ”字符串转换为数组{A,B,C}。


#### 9.7.12 字符串截取函数 fn:substring 函数
fn:substring函数用于截取字符串。它的参数、返回结果和描述如表9.28所示：

表9.28　fn:substring函数

<table cellspacing="0" cellpadding="0" width="353" border="1">
						<tbody>
							<tr>
								<td width="68">
								<p>参数</p>
								</td>
								<td width="285">
								<p>描述</p>
								</td>
							</tr>
							<tr>
								<td width="68">
								<p>string</p>
								</td>
								<td width="285">
								<p>源字符串。其类型必须为String类型</p>
								</td>
							</tr>
							<tr>
								<td width="68">
								<p>beginIndex</p>
								</td>
								<td width="285">
								<p>指定起始下标（值从0开始）。其类型必须为int类型</p>
								</td>
							</tr>
							<tr>
								<td width="68">
								<p>endIndex</p>
								</td>
								<td width="285">
								<p>指定结束下标（值从0开始）。其类型必须为int类型</p>
								</td>
							</tr>
							<tr>
								<td width="68">
								<p>返回结果</p>
								</td>
								<td width="285">
								<p>返回一个String类型的值</p>
								</td>
							</tr>
						</tbody>
					</table>


下面看一个示例。

${fn:substring("ABC","1","2")}<br>

截取结果为“ B ”。


#### 9.7.14 起始到定位截取字符串函数 fn:substringBefore 函数
fn:substringBefore函数允许截取源字符从开始到某个字符串。它的参数和fn:substringAfter函数相同，不同的是substring表示的是结束字符串。下面看一个示例。

    ${fn:substringBefore("ABCD","BC")}<br>

截取的结果为“ A ”。


#### 9.7.15 小写转换函数 fn:toLowerCase 函数
fn:toLowerCase函数允许将源字符串中的字符全部转换成小写字符。它只有一个表示源字符串的参数string，函数返回一个String类型的值。下面看一个示例。

    ${fn:toLowerCase("ABCD")}<br>

转换的结果为“ abcd ”。


#### 9.7.16大写转换函数 fn:toUpperCase 函数
fn:toUpperCase函数允许将源字符串中的字符全部转换成大写字符。它与fn:toLowerCase函数相同，也只有一个String参数，并返回一个String类型的值。下面看一个示例。

    ${fn:toUpperCase("abcd")}<br>

转换的结果为“ ABCD ”。


#### 9.7.17空格删除函数 fn:trim 函数
fn:trim函数将删除源字符串中结尾部分的“空格”以产生一个新的字符串。它与fn:toLowerCase函数相同，只有一个String参数，并返回一个String类型的值。下面看一个示例。

    ${fn:trim("AB C ")}D<br>

转换的结果为“ AB CD ”，注意，它将只删除词尾的空格而不是全部，因此“ B ”和“ C ”之间仍然留有一个空格。


参考文档<http://www.javawind.net/help/html/jstl_el.htm>
