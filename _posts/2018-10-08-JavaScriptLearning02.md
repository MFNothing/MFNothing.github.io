---
layout:     post
title:      "JavaScript 学习 2 ---- 在 HTML 中使用 JavaScript"
subtitle:   "JavaScript Learning"
date:       2018-09-11 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - JavaScript
---


## 在 HTML 中使用 JavaScript

### script 元素

```<script>``` 元素是向 HTML 页面中插入 JavaScript 的主要方法。

```<script>``` 元素定义了6个属性。

* async：可选。表示应该立即下载脚步，但不应妨碍页面中的其他操作，比如下载其他资源或等待加载其他脚本。只对外部脚本有效。
	* ```<script type="text/javascript" src="demo_async.js" async="async"></script>```
	* demo_async.js 会异步被执行
* defer：可选。表示脚本可以延迟到文档完全被解析和显示之后再执行。只对外部脚本文件有效。IE7 及更早版本对嵌入脚本也支持这个属性。
	* ```<script type="text/javascript" defer="defer">
alert(document.getElementById("p1").firstChild.nodeValue);
</script>```
* charset：可选。表示通过src属性指定的代码的字符集。由于大多数浏览器会忽略它的值，因此很少有人用。
	* ```<script type="text/javascript" src="myscripts.js" charset="UTF-8"></script>```
* src：可选。表示包含要执行代码的外部文件。
* type：可选。可以看出是 language 的替代属性；表示编写代码使用的脚步语言的内容类型（也称MIME 类型）。
* language：已废弃。

#### 标签的位置

按照惯例，所有```<script>```元素都应该放在页面的```<head>```元素中。这种做法的目的就是把所有外部文件（包括 CSS 文件和 JavaScript 文件）的引用都放在相同的地方。

```
<!DOCTYPE html>
<html>
	<head>
		<title> Example </title>
		<script type="text/javascript" src="1.js"> </script>
	</head>
	<body>
		<!-- 这里放内容 -->
	</body>
</html>
```

这意味着必须等到全部 JavaScript 代码都被下载、解析和执行完成以后，才能开始呈现页面的内容（浏览器在遇到```<body>```标签是才开始呈现内容）。所以对于很多 JavaScript 代码的页面来说，会在加载期间出现空白的现象。

为了避免这个问题，现代 Web 应用一般把全部 JavaScript 引用放在 ```<body>``` 元素中页面的内容后面。

```
<!DOCTYPE html>
<html>
	<head>
		<title> Example </title>
	</head>
	<body>
		<!-- 这里放内容 -->
		<script type="text/javascript" src="1.js"> </script>
	</body>
</html>
```



