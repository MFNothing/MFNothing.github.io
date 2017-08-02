---
layout:     post
title:      "Core Animation 01的学习"
subtitle:   "Learning Core Animation"
date:       2017-08-2 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Core Animation
---

## Core Animation

**Core Animation**是一个**复合引擎**，它的职责是尽可能快的**组合**屏幕上不同的**可视内容**，这个内容是被分解成独立的**图层**，存储于一个叫做**图层树**的体系之中。于是这个树形成了UIKit以及iOS应用程序当中你所能在屏幕上看到的一切基础。

### 图层和视图

#### 视图

**视图：**就是在屏幕上可以显示的一个矩形块（比如按钮、图片、视频或文字），它能够拦截类似于鼠标点击、触摸的手势等用户输入。

视图在**层级关系**中可以**相互嵌套**，一个视图可以**管理**它所有的子视图的位置。

在iOS中，所有视图都是从一个叫UIView的基类中派生而来，UIView可以处理触摸事件，可以支持基于Core Graphics绘图，可以做仿射变换（如旋转和缩放），或者简单的类似于滑动或者渐变的动画。

#### CALayer 图层

**CALayer** 类在概念上和UIView类似，同样也是一些被**层级关系树**管理的矩形块，同样也可以包含一些内容（图片、文本或者背景色），管理子图层的位置。它们有一些方法和属性来做动画和变换。与UIView最大的不同是CALayer**不处理**用户的交互。

即使它提供了一些方法判断是否一个触点在图层的范围之内。

#### 平行的层级关系

由UIView的层级关系形成的一种平行的CALayer层级关系

每一个UIView都有一个CALayer实例的**图层属性**，也就是所谓的backing layer，视图的职责就是创建**管理**这个图层，以确保当子视图在层级关系中添加或被移除的时候，他们关联的图层也同样对应在层级关系树当中有相同的操作。（这里关联的图层就是CALayer，当UIView中移除之后，会告诉CALayer你其中对应的图层也应该被移除了）

实际上这些背后关联的图层才是真正用来在屏幕上显示和做动画的，UIView仅仅是对它的封装，提供了一些iOS类似处理触摸的具体功能，以及Core Animation底层的高级接口。

**为什么iOS要基于UIView和CALayer提供两个平行的层级关系呢？**

原因是在于要做职责分离，这样也能避免很多重复的代码。如NSView和UIView，事件交互是不一样的，一个通过鼠标和键盘，一个通过手指触碰。而他们用到了CALayer。

实际上，这里并不是两个层级关系，而是四个，每一个都扮演不同的角色，除了视图层级和图层树之外，还存在呈现树和渲染树。后面会介绍。




### 图层的能力

CALayer可以说是UIView的内部实现细节。

我们可以通过CALayer做一些UIView没有暴露出来的功能。

* 阴影，圆角，带颜色的边框
* 3D变换
* 非矩形范围
* 透明遮罩
* 多级非线性动画


### 简单使用
创建一个工程然后在到Build Phases标签添加QuartzCore框架

```
#import "ViewController.h"
#import <QuartzCore/QuartzCore.h>
@interface ViewController ()
￼@end
@implementation ViewController
- (void)viewDidLoad
{
  	[super viewDidLoad];
   	UIView *smallView = [[UIView alloc] initWithFrame:CGRectMake(50, 50, 200, 200)];
    smallView.backgroundColor = [[UIColor yellowColor] colorWithAlphaComponent:0.3];
    CALayer *blueLayer = [CALayer layer];
    blueLayer.backgroundColor = [[UIColor blueColor] colorWithAlphaComponent:0.3].CGColor;
    blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    [smallView.layer addSublayer:blueLayer];
    UIView *ssView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 50, 50)];
    ssView.backgroundColor = [[UIColor redColor] colorWithAlphaComponent:0.3];
    [smallView addSubview:ssView];
    [self.view addSubview:smallView];
}
@end
```

效果图
![](/img/in-mpost/Core-Animation-01/rendering.png)

层次结构
![](/img/in-mpost/Core-Animation-01/hierarchy.png)

一个视图只有一个相关联的图层（自动创建），同时它也可以支持添加无数多个子图层。你可以显示创建一个单独的图层，并且把它直接添加到视图关联图层的子图层。尽管可以这样添加图层，但往往我们只是见简单地处理视图，他们关联的图层并不需要额外地手动添加子图层。

使用图层关联的视图而不是CALayer的好处在于，你能在使用所有CALayer底层特性的同时，也可以使用UIView的高级API（比如自动排版，布局和事件处理）。
