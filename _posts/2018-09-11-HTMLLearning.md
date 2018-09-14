---
layout:     post
title:      "HTML 学习"
subtitle:   "HTML Learning"
date:       2018-09-11 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - HTML
---


## HTML

### 什么是HTML

#### HTML 

**HTML 是用来描述网页的一种语言。**

* HTML 指的是超文本标记语言 (Hyper Text Markup Language)
* HTML 不是一种编程语言，而是一种标记语言 (markup language)
* 标记语言是一套标记标签 (markup tag)
* HTML 使用标记标签来描述网页

#### HTML 标签

**HTML 标记标签通常被称为 HTML 标签 (HTML tag)。**

* HTML 标签是由尖括号包围的关键词，比如`<html>`
* HTML 标签通常是成对出现的，比如`<b>`和`</b>`
* 标签对中的第一个标签是开始标签，第二个标签是结束标签
* 开始和结束标签也被称为开放标签和闭合标签

### HTML 基础

#### 元素语法

* HTML 元素以开始标签起始，如`<h1>`
* HTML 元素以结束标签终止，如`</h1>`
* 元素的内容是开始标签与结束标签之间的内容，如`<h1>xxx</h1>`，xxx就是元素的内容
* 某些 HTML 元素具有空内容（empty content）
* 空元素在开始标签中进行关闭（以开始标签的结束而结束）
	* 没有内容的 HTML 元素被称为空元素。空元素是在开始标签中关闭的。
	* 在开始标签中添加斜杠，比如 <br />，是关闭空元素的正确方法。
* 大多数 HTML 元素可拥有属性，如可选属性、标准属性和事件属性。
* HTML 标签对大小写不敏感：<P> 等同于 <p>。许多网站都使用大写的 HTML 标签。

```
<html>
	<body>
		<h1>标题</h1>
		<p>段落</p>
		<a href="http://xxx">链接</>
		<img src="http://xxx" />
	</body>
</html>
```

#### 属性

* HTML 标签可以拥有属性。属性提供了有关 HTML 元素的更多的信息。
* 属性总是以名称/值对的形式出现，比如：name="value"。
 * 属性和属性值对大小写不敏感。
 * 推荐小写属性和属性值。
* 属性总是在 HTML 元素的开始标签中规定。
* 始终为属性值加引号。
 * 属性值应该始终被包括在引号内。双引号是最常用的，不过使用单引号也没有问题。
 * 在某些个别的情况下，比如属性值本身就含有双引号，那么您必须使用单引号，如`name='some"new"'`

适用于大多数HTML元素的属性

|属性|值|描述|
|---|---|---|
|class|classname|规定元素的类名(classname)|
|id|id|规定元素的唯一id|
|style|style_definition|规定元素的行内样式(inline style)|
|title|text|规定元素的额外信息(可在工具提示中显示)|

#### 标题 

**`<h1>最大字体标题</h1>`至`<h6>最小能用标题</h6>`**

* 浏览器会自动地在标题的前后添加空行。
* 默认情况下，HTML 会自动地在块级元素前后添加一个额外的空行，比如段落、标题元素前后。
* 请确保将 HTML heading 标签只用于标题。不要仅仅是为了产生粗体或大号的文本而使用标题。
* 搜索引擎使用标题为您的网页的结构和内容编制索引。
* 因为用户可以通过标题来快速浏览您的网页，所以用标题来呈现文档结构是很重要的。

`<hr />`标签在 HTML 页面中创建水平线。用于分割内容。

```
<html>
	<body>
		<h1>大标题</h1>
		<hr/>
	</body>
</html>
```

#### 段落 

**`<p>段落</p>`**

* 浏览器会自动地在段落的前后添加空行。但是只是为了添加空行，可以使用`<br />`替代。
* 如果在段落中不使用新段落的情况下进行换行，请使用`<br />`
* 当显示页面时，浏览器会移除源代码中多余的空格和空行。段落中所有连续的空格或空行都会被算作一个空格。需要注意的是，HTML 代码中的所有连续的空行（换行）也被显示为一个空格。

```
<html>
	<body>
		<p> 段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字 </p>
		<p> 段落文字  段落文字 
		<!--多余空格和空行--->
		
		
		
		
		<!--多余空格和空行--->
		 段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字 </p>
		<p> 段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字  段落文字 </p>
	</body>
</html>
```

#### 样式 sytle属性(内联样式)

样式是 HTML 4 引入的，它是一种新的首选的改变 HTML 元素样式的方式。通过 HTML 样式，能够通过使用 style 属性直接将样式添加到 HTML 元素，或者间接地在独立的样式表中（CSS 文件）进行定义。

* sytle属性：提供了一种改变所有 HTML 元素的样式的通用方法。
* 背景颜色 `style="background-color:yellow"`
* 字体、颜色和尺寸 `style="font-family:arial;color:red;font-size:20px;"`
* 文本对齐 `style="text-align:center"`

```
<html>
	<body style="background-color:yellow">
		<h1 style="font-family:arial;color:blue;">大标题</h1>
		<p style="text-align:center;font-size:15px;background-color:red">段落</p>
	</body>
</html>
```

#### 引用 

**`<q>` `<blockquote>` `<abbr>` `<address>`**

一般都在段落中使用。

* `<q>`元素定义短的引用，浏览器通常会为`<q>`元素包围引号。
* `<blockquote>`元素定义被引用的节，浏览器通常会对`<blockquote>`元素进行缩进处理。
* `<abbr>`元素定义缩写或首字母缩略语，对缩写进行标记能够为浏览器、翻译系统以及搜索引擎提供有用的信息。
* `<address>`元素定义文档或文章的联系信息（作者/拥有者），此元素通常以斜体显示。大多数浏览器会在此元素前后添加折行。

#### CSS

当浏览器读到一个样式表，它就会按照这个样式表来对文档进行格式化。有以下三种方式来插入样式表：

* 外部样式表
 * 当样式需要被应用到很多页面的时候，外部样式表将是理想的选择。使用外部样式表，你就可以通过更改一个文件来改变整个站点的外观。
 * 通过引入一个css文件来改变样式 `<link rel="stylesheet" type="text/css" href="/html/csstest1.css" >`
* 内部样式表
 * 当单个文件需要特别样式时，就可以使用内部样式表。你可以在 head 部分通过 `<style>` 标签定义内部样式表。 
 * `<style type="text/css"> body {background-color: red} p {margin-left: 20px} </style>` 这里修改了body标签和p标签的样式。
* 内联样式
 * 当特殊的样式需要应用到个别元素时，就可以使用内联样式。 使用内联样式的方法是在相关的标签中使用样式属性。样式属性可以包含任何 CSS 属性。以下实例显示出如何改变段落的颜色和左外边距。
 * `<p style="color: red; margin-left: 20px">xxx</p>`

#### 链接 

**`<a href="http://xxx">链接名</a>`**

* 超链接可以是一个字，一个词，或者一组词，也可以是**一幅图像**，您可以点击这些内容来跳转到新的文档或者当前文档中的某个部分。
* target 属性
 * 使用 Target 属性，你可以定义被链接的文档在何处显示。
 * 	可选字段 `_blank` `_parent` `_self` `_top` `framename`，注意`_top`可以跳出框架，如果你的页面被指定到某个框架内。
 *  `framename`主要用于跳转指定name属性的框架，比如`<iframe src="demo_iframe.html" name="iframe_a"></iframe>`，这个`<iframe>` 指定 name 为 iframe_a，那么我们可以这样写链接 `<a href="http://xxx" target="iframe_a">xxx</a>`这样我们点击链接的时候，这个iframe就会跳转到链接的地址。
 *  `<a href="http://xxx" target="_self">显示内容</a>`
* name 属性
 * 使用 name 属性创建 HTML 页面中的书签。书签不会以任何方式显示。
 * name 属性规定锚（anchor）的名称。当使用命名锚（named anchors）时，我们可以创建直接跳至该命名锚（比如页面中某个小节）的链接，这样使用者就无需不停地滚动页面来寻找他们需要的信息了。
 * 命名锚的语法: `<a name="tips">锚（显示在页面上的文本）</a>`
 * 跳转锚的方式：同一文档内 `<a href="#tips">有用的提示</a>`; 其他页面内跳转 `<a href="http://xxxx#tips">有用的提示</a>`
 * 如果找不到锚，不会出错，会跳到文档顶部。
 
#### 图片 

**`<img src="http://xxx"/>`**

* 插入gif和非gif图片都是一样的方式 `<img src="file/xxx.gif"/>`

#### 表格 

**`<table>` `<tr>` `<td>` `<th>`**

* 表格由 `<table>` 标签来定义。每个表格均有若干行（由 `<tr>` 标签定义），每行被分割为若干单元格（由 `<td>` 标签定义）。字母 td 指表格数据（table data），即数据单元格的内容。数据单元格可以包含文本、图片、列表、段落、表单、水平线、表格等等。
* `<th>` 设置表头，效果字体加粗
* 跨行(单元格占用多行) `<th rowspan="2">单元格</th>`； 垮列（单元格占用多列）`<td colspan="3">单元格</td>`


```
<html>

<body>

<h4>表头：</h4>
<table border="1">
<tr>
  <th>姓名</th>
  <th colspan="2">电话</th>
</tr>
<tr>
  <td>Bill Gates</td>
  <td>555 77 854</td>
  <td>555 77 855</td>
</tr>
</table>

<h4>垂直的表头：</h4>
<table border="1">
<tr>
  <th>姓名</th>
  <td>Bill Gates</td>
</tr>
<tr>
  <th rowspan="2">电话</th>
  <td>555 77 854</td>
</tr>
<tr>
  <td>555 77 855</td>
</tr>
</table>

</body>
</html>
```

#### 列表 

**`<ol>` `<ul>` `<li>` `<dl>`**

* 无序列表使用 `<ul>`， unordered list
 * 无序列表是一个项目的列表，此列项目使用粗体圆点（典型的小黑圆圈）进行标记。
* 有序列表使用 `<ol>`， ordered list
 * 有序列表也是一列项目，列表项目使用数字进行标记
* 定义列表 `<dl>` define list, `<dt>` start, `dd` end
 * 自定义列表不仅仅是一列项目，而是项目及其注释的组合。
 * 自定义列表以 `<dl>` 标签开始。每个自定义列表项以 `<dt>` 开始。每个自定义列表项的定义以 `<dd>` 开始。
 * 定义列表的列表项内部可以使用段落、换行符、图片、链接以及其他列表等等。
* 列表元素 `<li>`
* 使用 type 属性改变无序列表和有序列表的样式。
* 列表可以嵌套

```
// 无序列表
<ul>
<li>Coffee</li>
<li>Milk</li>
</ul>

// 有序列表
<ol>
<li>Coffee</li>
<li>Milk</li>
</ol>

// 定义列表
<dl>
   <dt>计算机</dt>
   <dd>用来计算的仪器 ... ...</dd>
   <dt>显示器</dt>
   <dd>以视觉方式显示信息的装置 ... ...</dd>
</dl>
```

#### 块

大多数 HTML 元素被定义为块级元素或内联元素。

**块元素与内联函数的区别**

* 块元素
 * 块级元素在浏览器显示时，通常会以新行来开始（和结束）。
 * 例如 `<h1>` `<p>` `<ul>` `<table>`
* 内联元素
 * 内联元素在显示时通常不会以新行开始。
 * 例如 `<td>` `<a>` `<img>`
 
**`<div>` 元素**

* HTML`<div> `元素是块级元素，它是可用于组合其他 HTML 元素的容器。
* `<div>`是一个块级元素，也就是说，浏览器通常会在 div 元素前后放置一个换行符。
* 如果与 CSS 一同使用，<div> 元素可用于对大的内容块设置样式属性。

```
<div style="color:#00FF00">
  <h3>This is a header</h3>
  <p>This is a paragraph.</p>
</div>
```

**`<span>` 元素**

* HTML `<span>` 元素是内联元素，可用作文本的容器。
* 当与 CSS 一同使用时，`<span>` 元素可用于为部分文本设置样式属性。

```
<!DOCTYPE html>
<html>
<head>
<style>
  .red {color:red;}
</style>
</head>
<body>

<h1>My <span class="red">Important</span> Heading</h1>

</body>
</html>
```

#### 类

* 对 HTML 进行分类（设置类），使我们能够为元素的类定义 CSS 样式。
* 为相同的类设置相同的样式，或者为不同的类设置不同的样式。

```
<!DOCTYPE html>
<html>
<head>
<style>
.cities {
    background-color:black;
    color:white;
    margin:20px;
    padding:20px;
} 
</style>
</head>

<body>

<div class="cities">
<h2>London</h2>
<p>
London is the capital city of England. 
It is the most populous city in the United Kingdom, 
with a metropolitan area of over 13 million inhabitants.
</p>
</div> 

</body>
</html>
```

#### 布局

HTML5 提供的新语义元素定义了网页的不同部分

| 语义元素	|		定义						|
|	---		|				:-------:			|
| header 	|	定义文档或节的页眉				|
| nav 		|	定义文档或节的页眉				|
| section |	定义文档或节的页眉				|
| article |	定义独立的自包含文章 			|
| aside 	|	定义内容之外的内容（比如侧栏）	|
| footer 	| 	定义文档或节的页脚				|
| details |	定义额外的细节					|
| summary |	定义 details 元素的标题		|

```
<!DOCTYPE html>
<html>

<head>
<style>
header {
    background-color:black;
    color:white;
    text-align:center;
    padding:5px;	 
}
nav {
    line-height:30px;
    background-color:#eeeeee;
    height:300px;
    width:100px;
    float:left;
    padding:5px;	      
}
section {
    width:350px;
    float:left;
    padding:10px;	 	 
}
footer {
    background-color:black;
    color:white;
    clear:both;
    text-align:center;
    padding:5px;	 	 
}
</style>
</head>

<body>

<header>
<h1>City Gallery</h1>
</header>

<nav>
London<br>
Paris<br>
Tokyo<br>
</nav>

<section>
<h1>London</h1>
<p>
London is the capital city of England. It is the most populous city in the United Kingdom,
with a metropolitan area of over 13 million inhabitants.
</p>
<p>
Standing on the River Thames, London has been a major settlement for two millennia,
its history going back to its founding by the Romans, who named it Londinium.
</p>
</section>

<footer>
Copyright W3Schools.com
</footer>

</body>
</html>

```

#### 框架 

**`<frame>` `<frameset>`**

* 通过使用框架，你可以在同一个浏览器窗口中显示不止一个页面。每份HTML文档称为一个框架，并且每个框架都独立于其他的框架。
* 通过使用框架结构标签(`<frameset>`)来分配显示页面在浏览器中的配比。
* 如果浏览器无法支持框架可以通过`<noframes>`标签内添加内容来提示用户无法使用框架。
 * `<noframes> <body>您的浏览器无法处理框架！</body> </noframes>`

```
// 多列展示页面
<frameset cols="25%,75%">
   <frame src="frame_a.htm">
   <frame src="frame_b.htm">
</frameset>
// 多行展示页面
<frameset rows="25%,50%,25%">
  <frame src="/example/html/frame_a.html">
  <frame src="/example/html/frame_b.html">
  <frame src="/example/html/frame_c.html">
</frameset>
// 混合展示
<frameset rows="50%,50%">

<frame src="/example/html/frame_a.html">

<frameset cols="25%,75%">
<frame src="/example/html/frame_b.html">
<frame src="/example/html/frame_c.html">
</frameset>

</frameset>
```

#### 内联框架 

**`<iframe>`**

* `<iframe src="URL"></iframe>`
* 设置高度和宽度
 * height 和 width 属性用于规定 iframe 的高度和宽度。
 * 属性值的默认单位是像素，但也可以用百分比来设定（比如 "80%"）。
 * `<iframe src="demo_iframe.htm" width="200" height="200"></iframe>`
* 删除边框
 * frameborder 属性规定是否显示 iframe 周围的边框。
 * 设置属性值为 "0" 就可以移除边框。
 * `<iframe src="demo_iframe.htm" frameborder="0"></iframe>`
* 使用 iframe 作为链接的目标
 * iframe 可用作链接的目标（target）。 如 `<a href="http://xxx" target="iframe_a">xxxx</a>`
 * 链接的 target 属性必须引用 iframe 的 name 属性。如 `<iframe src="demo_iframe.htm" name="iframe_a"></iframe>`

#### 头部元素

* `<head>` 元素是所有头部元素的容器。`<head>` 内的元素可包含脚本，指示浏览器在何处可以找到样式表，提供元信息，等等。
* 可以添加到 head 部分的标签：`<title>`、`<base>`、`<link>`、`<meta>`、`<script>` 以及 `<style>`。

**`<title>` 元素**

* 定义浏览器工具栏中的标题，就是关闭网页按钮旁边显示的标题。
* 提供页面被添加到收藏夹时显示的标题
* 显示在搜索引擎结果中的页面标题

```
<!DOCTYPE html>
<html>
<head>
<title>Title of the document</title>
</head>

<body>
The content of the document......
</body>

</html>
```

**`<base>` 元素**

* `<base>` 标签为页面上的所有链接规定默认地址或默认目标（target）

```
<!DOCTYPE html>
<html>
<head>
<base href="http://www.w3school.com.cn/images/" />
<base target="_self" />
</head>

<body>
<a href=".gif">The content of the document......</a>
</body>

</html>
```

这里我们定义了一个链接设置并链接的地址为.gif，由于我们设置`<base>`标签，所以它最终跳转的地址是
`http://www.w3school.com.cn/images/.gif`

这里无论我们设置链接地址为什么，都会拼接到`http://www.w3school.com.cn/images/`后面。

只是当我们设置链接target会覆盖默认的target，如果不设置会使用默认`<base>`标签中的target。

**`<link>` 元素**

* `<link>` 标签定义文档与外部资源之间的关系。
 * **rel** 属性： 规定当前文档与被链接文档之间的关系。
 * **type** 属性： 规定被链接文档的 MIME 类型。
 * **href** 属性：	规定被链接文档的位置。

```
<head>
<link rel="stylesheet" type="text/css" href="mystyle.css" />
</head>
```

**`<style>` 元素**

* `<style>` 标签用于为 HTML 文档定义样式信息。

```
<head>
<style type="text/css">
body {background-color:yellow}
p {color:blue}
</style>
</head>
```

**`<meta>` 元素**

* `<meta>` 标签提供关于 HTML 文档的元数据。元数据不会显示在页面上，但是对于机器是可读的。
 * 元数据（metadata）是关于数据的信息。
 * 典型的情况是，meta 元素被用于规定页面的描述、关键词、文档的作者、最后修改时间以及其他元数据。
 * `<meta>` 标签始终位于 head 元素中。
 * 元数据可用于浏览器（如何显示内容或重新加载页面），搜索引擎（关键词），或其他 web 服务。
* 一些搜索引擎会利用 meta 元素的 name 和 content 属性来索引您的页面。
 * meta 元素定义页面的描述 `<meta name="description" content="Free Web tutorials on HTML, CSS, XML" />`
 * meta 元素定义页面的关键词 `<meta name="keywords" content="HTML, CSS, XML" />`

 **`<script>` 元素**
 
 * `<script> `标签用于定义客户端脚本，比如 JavaScript。

#### 字符实体

因为在HTML中，某些字符是预留的，不能够直接被我们使用显示在屏幕上，所以我们需要字符实体。

* 实体名称对大小写敏感！


|显示结果	|描述				|实体名称		| 实体编号|
|---		|---				|---			|---|
| 			|空格 				|	`&nbsp;`	|`&#160;`|
|<			|小于号				|`	&lt;`		|`&#60;`|
|>			|大于号				|`&gt;`		|`&#62;`|
|&			|和号				|`&amp;`		|`&#38;`|
|"			|引号				|`&quot;`		|`&#34;`|
|'			|撇号 				|`&apos;` (IE不支持)|`&#39;`|
|￠			|分（cent）		|`&cent;`		|`&#162;`|
|£			|镑（pound）		|`&pound;`	|`&#163;`|
|¥			|元（yen）			|`&yen;`		|`&#165;`|
|€			|欧元（euro）		|`&euro;`		|`&#8364;`|
|§			|小节				|`&sect;`		|`&#167;`|
|©			|版权（copyright）|`&copy;`		|`&#169;`|
|®			|注册商标			|`&reg;`		|`&#174;`|
|™			|商标				|`&trade;`	|`&#8482;`|
|×			|乘号				|`&times;`	|`&#215;`|
|÷			|除号				|`&divide;`	|`&#247;`|

#### URL

* URL(Uniform Resource Locator/统一资源定位器) 也被称为网址。
* URL 可以由单词组成，比如 “w3school.com.cn”，或者是因特网协议（IP）地址：192.168.1.253。大多数人在网上冲浪时，会键入网址的域名，因为名称比数字容易记忆。
* 语法规则
 * `scheme://host.domain:port/path/filename`
 * `http://www.w3school.com.cn/html/index.asp`
 * scheme - 定义因特网服务的类型。最常见的类型是 http
 * host - 定义域主机（http 的默认主机是 www）
 * domain - 定义因特网域名，比如 w3school.com.cn
 * :port - 定义主机上的端口号（http 的默认端口号是 80）
 * path - 定义服务器上的路径（如果省略，则文档必须位于网站的根目录中）。
 * filename - 定义文档/资源的名称
* URL 编码
 * URL 只能使用 ASCII 字符集来通过因特网进行发送。
 * 由于 URL 常常会包含 ASCII 集合之外的字符，URL 必须转换为有效的 ASCII 格式。
 * URL 编码使用 "%" 其后跟随两位的十六进制数来替换非 ASCII 字符。
 * URL 不能包含空格。URL 编码通常使用 + 来替换空格。

#### 表单 

**`<form>`**

HTML 表单用于收集用户输入。`<form> `元素定义 HTML 表单。

* HTML 表单包含表单元素。表单元素指的是不同类型的 input 元素：文本输入、复选框、单选按钮、提交按钮等等。
* 文本输入 `<input type="text">`
 * 需要注意当提交表单的时候，必须有 name 属性，且可以设置默认填充值
 * `<input type="text" name="get请求中&左边显示内容" value="默认填充值">`
* 单选按钮输入 `<input type="radio">`
 * 需要注意只有相同 name 属性的才有单选效果，且带有checked属性的是默认选中的
 * 然后这个默认只是一个按钮，没有文字的，添加文字需要在标签后面加
 * `<input type="radio" name="radioname" value="get请求中&右边显示的内容" checked>`
* 提交按钮 `<input type="submit">`
 * 定义用于向表单处理程序（form-handler）提交表单的按钮。
 * 表单处理程序通常是包含用来处理输入数据的脚本的服务器页面。
 * 表单处理程序在表单的 action 属性中指定。
 * `<input type="submit" value="按钮名">`
* Action 属性
 * action 属性定义在提交表单时执行的动作。
 * 通常，表单会被提交到 web 服务器上的网页。
 * `<form action="action_page.php">`
* Method 属性
 * method 属性规定在提交表单时所用的 HTTP 方法（GET 或 POST）
 * `<form action="action_page.php" method="GET">`
 * `<form action="action_page.php" method="POST">`
 * 需要注意的是，表单数据在页面地址栏中是可见的：`action_page.php?firstname=Mickey&lastname=Mouse`，并且 GET 最适合少量数据的提交。浏览器会设定容量限制。
 * POST 的安全性更加，因为在页面地址栏中被提交的数据是不可见的。
* Name 属性
 * 如果要正确地被提交，每个输入字段必须设置一个 name 属性。就是表单元素。否则提交不上去。
* 用 `<fieldset>` 组合表单数据
 * `<legend>` 元素为 `<fieldset>` 元素定义标题。
 * 就是把在其中的表元素包起来。
* Form 可选属性
 * `<form action="action_page.php" method="GET" target="_blank" accept-charset="UTF-8" ectype="application/x-www-form-urlencoded" autocomplete="off" novalidate>.form elements.</form>`


|属性	|描述|
|---|---|
|accept-charset	|规定在被提交表单中使用的字符集（默认：页面字符集）。|
|action	|规定向何处提交表单的地址（URL）（提交页面）。|
|autocomplete	|规定浏览器应该自动完成表单（默认：开启）。|
|enctype	|规定被提交数据的编码（默认：url-encoded）。|
|method	|规定在提交表单时所用的 HTTP 方法（默认：GET）。|
|name	|规定识别表单的名称（对于 DOM 使用：document.forms.name）。|
|novalidate	|规定浏览器不验证表单。|
|target	|规定 action 属性中地址的目标（默认：_self）。|


#### 表单元素
 
* `<input>` 元素，主要根据 type 属性变化多个形态。
 * `<input type="checkbox">` 定义复选框。
 * HTML5 增加了多个新的输入类型
 * color
 * date
 * datetime
 * datetime-local
 * email
 * month
 * number
 * range
 * search
 * tel
 * time
 * url
 * week
* `<select>` 元素（下拉列表）
 * `<option>` 元素定义待选择的选项。
 * 列表通常会把首个选项显示为被选选项。
 * 您能够通过添加 selected 属性来定义预定义选项。
 * `<option value="fiat" selected>Fiat</option>`
 * `<select name = "name"> <option value = "flat"> Flat </option> </select>`
* `<textarea>` 元素
 * `<textarea>` 元素定义多行输入字段（文本域）
 * `<textarea name="message" rows="10" cols="30"> text </textarea>`
* `<button>` 元素
 * `<button type="button" onclick="alert('Hello World!')">Click Me!</button>`
* HTML5 `<datalist>` 元素
 * `<datalist>` 元素为 `<input>` 元素规定预定义选项列表。
 * 用户会在他们输入数据时看到预定义选项的下拉列表。
 * ` <input>` 元素的 list 属性必须引用 `<datalist>` 元素的 id 属性。
 * `<input list="browsers"> <datalist id="browsers"> <option value="Safari"> </datalist> </form>`
