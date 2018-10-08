---
layout:     post
title:      "JavaScript 学习 1 ---- JavaScript 简介"
subtitle:   "JavaScript Learning"
date:       2018-09-11 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - JavaScript
---


## JavaScript 简介

开始 JavaScript 只是为了处理简单验证而产生的语言。现在 JavaScript 具备了与浏览器窗口及其内容等几乎所有方面交互的能力。

### JavaScript 实现

由三个部分组成 

* 核心（ECMAScript），描述了该语言的语法和基本对象。
* 文档对象模型（DOM），描述了处理网页内容的方法和接口。
* 浏览器对象模型（BOM），描述了与浏览器进行交互的方法和接口。

#### ECMAScript

ECMA组织发布262号标准文件（ECMA-262）的第一版，规定了浏览器脚本语言的标准，并将这种语言称为ECMAScript。

* ECMAScript 与 Web 浏览器没有依赖关系。
* Web 浏览器只是 ECMAScript 实现可能的宿主环境之一。宿主环境不仅提供基本的 ECMAScript 的实现，同事也会提供该语言的拓展，以便语言与环境之间对接交互。其他宿主环境包括 Node 和 Adobe Flash。
* ECMAScript 规定了这门语言组成部分：语法、类型、语句、关键字、保留字、操作符和对象。
* JavaScript 和 Adobe ActionScript 同样实现了 ECMAScript。

#### 文档对象模型（DOM）

文档对象模型（DOM, Document Object Model）是针对 XML 但是经过拓展用于 HTML 的应用程序编程接口（API， Application Programming Interface）。

* 借助 DOM 提供的API，开发人员可以轻松自如地删除、添加、替换或修改任何节点。
* 为什么使用 DOM？无需重新加载网页就可以修改其外观和内容。
* DOM 级别，DOM1 级由两个模块组成：DOM 核心和 DOM HTML。
 * DOM 核心规定的是如何映射基于XML的文档结构，一边简化对文档中的任意部分的访问和操作。
 * DOM HTML 模块则是在 DOM 核心的基础上加以拓展，添加针对 HTML 的对象和方法。
 * DOM2 级拓展了 鼠标和用户界面事件、遍历和操作文档树等。

#### 浏览器对象模型（BOM）

浏览器对象模型（BOM，Browser Object Model），开发人员可以使用 BOM 控制浏览器显示页面之外的部分。HTML 5之后才支持其大部分功能。BOM 公共特性实现的效果，因浏览器而异。

* 从根本上来讲，BOM 只处理浏览器窗口和框架，但是人们习惯也把所有针对浏览器的 JavaScript 拓展算作 BOM 的一部分。下面是一些这样的拓展：
 * 弹出浏览器窗口的功能。
 * 移动、缩放和关闭浏览器窗口的功能。
 * 提供浏览器详细信息的 navigator 对象。
 * 提供浏览器所加载页面的详细信息的 location 对象。
 * 提供用户显示器分辨率详细信息的 screen 对象。
 * 对 cookies 的支持。
