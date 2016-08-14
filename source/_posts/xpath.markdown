---
layout: post
title: "xpath 详解"
date: 2012-10-20 10:43
comments: true
categories: selenium
tags: [ webdriver, selenium, xpath, DOM ]
---
<div id="intro">
<p><strong>XPath 使用路径表达式来选取 XML 文档中的节点或节点集。节点是通过沿着路径 (path) 或者步 (steps) 来选取的。</strong></p>
</div>


<div>
<h2>XML 实例文档</h2>
<p>我们将在下面的例子中使用这个 XML 文档。</p>
<pre>&lt;?xml version=&quot;1.0&quot; encoding=&quot;ISO-8859-1&quot;?&gt;

&lt;bookstore&gt;

&lt;book&gt;
  &lt;title lang=&quot;eng&quot;&gt;Harry Potter&lt;/title&gt;
  &lt;price&gt;29.99&lt;/price&gt;
&lt;/book&gt;

&lt;book&gt;
  &lt;title lang=&quot;eng&quot;&gt;Learning XML&lt;/title&gt;
  &lt;price&gt;39.95&lt;/price&gt;
&lt;/book&gt;

&lt;/bookstore&gt;</pre>
</div>
<!--more-->

<div>
<h2>选取节点</h2>

<p>XPath 使用路径表达式在 XML 文档中选取节点。节点是通过沿着路径或者 step 来选取的。</p>

<h3>下面列出了最有用的路径表达式：</h3>

<table class="dataintable">
<tr>
<th style="width:25%;">表达式</th>
<th>描述</th>
</tr>

<tr>
<td>nodename</td>
<td>选取此节点的所有子节点。</td>
</tr>

<tr>
<td>/</td>
<td>从根节点选取。</td>
</tr>

<tr>
<td>//</td>
<td>从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置。</td>
</tr>

<tr>
<td>.</td>
<td>选取当前节点。</td>
</tr>

<tr>
<td>..</td>
<td>选取当前节点的父节点。</td>
</tr>

<tr>
<td>@</td>
<td>选取属性。</td>
</tr>
</table>

<h3>实例</h3>

<p>在下面的表格中，我们已列出了一些路径表达式以及表达式的结果：</p>

<table class="dataintable">
<tr>
<th style="width:25%;">路径表达式</th>
<th>结果</th>
</tr>

<tr>
<td>bookstore</td>
<td>选取 bookstore 元素的所有子节点。</td>
</tr>

<tr>
<td>/bookstore</td>
<td>
<p>选取根元素 bookstore。</p>
<p>注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！</p>
</td>
</tr>

<tr>
<td>bookstore/book</td>
<td>选取属于 bookstore 的子元素的所有 book 元素。</td>
</tr>

<tr>
<td>//book</td>
<td>选取所有 book 子元素，而不管它们在文档中的位置。</td>
</tr>

<tr>
<td>bookstore//book</td>
<td>选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。</td>
</tr>

<tr>
<td>//@lang</td>
<td>选取名为 lang 的所有属性。</td>
</tr>
</table>
</div>


<div>
<h2>谓语（Predicates）</h2>

<p>谓语用来查找某个特定的节点或者包含某个指定的值的节点。</p>
<p>谓语被嵌在方括号中。</p>

<h3>实例</h3>

<p>在下面的表格中，我们列出了带有谓语的一些路径表达式，以及表达式的结果：</p>

<table class="dataintable">
<tr>
<th style="width:35%;">路径表达式</th>
<th>结果</th>
</tr>

<tr>
<td>/bookstore/book[1]</td>
<td>选取属于 bookstore 子元素的第一个 book 元素。</td>
</tr>

<tr>
<td>/bookstore/book[last()]</td>
<td>选取属于 bookstore 子元素的最后一个 book 元素。</td>
</tr>

<tr>
<td>/bookstore/book[last()-1]</td>
<td>选取属于 bookstore 子元素的倒数第二个 book 元素。</td>
</tr>

<tr>
<td>/bookstore/book[position()&lt;3]</td>
<td>选取最前面的两个属于 bookstore 元素的子元素的 book 元素。</td>
</tr>

<tr>
<td>//title[@lang]</td>
<td>选取所有拥有名为 lang 的属性的 title 元素。</td>
</tr>

<tr>
<td>//title[@lang='eng']</td>
<td>选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。</td>
</tr>

<tr>
<td>/bookstore/book[price&gt;35.00]</td>
<td>选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。</td>
</tr>

<tr>
<td>/bookstore/book[price&gt;35.00]/title</td>
<td>选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。</td>
</tr>
</table>
</div>


<div>
<h2>选取未知节点</h2>

<p>XPath 通配符可用来选取未知的 XML 元素。</p>

<table class="dataintable">
<tr>
<th style="width:25%;">通配符</th>
<th>描述</th>
</tr>

<tr>
<td>*</td>
<td>匹配任何元素节点。</td>
</tr>

<tr>
<td>@*</td>
<td>匹配任何属性节点。</td>
</tr>

<tr>
<td>node()</td>
<td>匹配任何类型的节点。</td>
</tr>
</table>

<h3>实例</h3>

<p>在下面的表格中，我们列出了一些路径表达式，以及这些表达式的结果：</p>

<table class="dataintable">
<tr>
<th style="width:25%;">路径表达式</th>
<th>结果</th>
</tr>

<tr>
<td>/bookstore/*</td>
<td>选取 bookstore 元素的所有子元素。</td>
</tr>

<tr>
<td>//*</td>
<td>选取文档中的所有元素。</td>
</tr>

<tr>
<td>//title[@*]</td>
<td>选取所有带有属性的 title 元素。</td>
</tr>
</table>
</div>


<div>
<h2>选取若干路径</h2>

<p>通过在路径表达式中使用“|”运算符，您可以选取若干个路径。</p>

<h3>实例</h3>

<p>在下面的表格中，我们列出了一些路径表达式，以及这些表达式的结果：</p>

<table class="dataintable">
<tr>
<th style="width:35%;">路径表达式</th>
<th>结果</th>
</tr>

<tr>
<td>//book/title | //book/price</td>
<td>选取 book 元素的所有 title 和 price 元素。</td>
</tr>

<tr>
<td>//title | //price</td>
<td>选取文档中的所有 title 和 price 元素。</td>
</tr>

<tr>
<td>/bookstore/book/title | //price</td>
<td>选取属于 bookstore 元素的 book 元素的所有 title 元素，以及文档中所有的 price 元素。</td>
</tr>
</table>
</div>

