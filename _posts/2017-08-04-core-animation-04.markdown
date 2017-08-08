---
layout:     post
title:      "Core Animation 04的学习————视觉效果"
subtitle:   "Learning Core Animation"
date:       2017-08-04 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Core Animation
---


## 视觉效果

### 圆角 cornerRadius

cornerRadius 属性控制着图层的曲率。默认为0表示直角，可以为任何值，但是小于0都是直角。这个值只影响背景颜色而不影响背景图片或是子图层。

```
layer.cornerRadius = 20.0f;
layer.contentsGravity = kCAGravityCenter;
```
效果图
![](/img/in-mpost/Core-Animation-04/cornerRadius.png)

不过如果把maskToBounds设置为YES的话，图层里面的所有东西都会被截取。

### 图层边框 stroke

CALayer中使用borderWidth和borderColor属性共同定义图层边的绘制样式。

**borderWidth**是以点为单位的定义边框粗细的浮点数，默认为0。

**borderColor**定义了边框的颜色，默认为黑色。

**borderColor**是CGColorRef类型，而不是UIColor，所以它不是Cocoa的内置对象。不过呢，你肯定也清楚图层引用了borderColor，虽然属性声明并不能证明这一点。CGColorRef在引用/释放时候的行为表现得与NSObject极其相似。但是Objective-C语法并不支持这一做法，所以CGColorRef属性即便是强引用也只能通过assign关键字来声明。

```
layer.borderWidth = 5;
layer.borderColor = [UIColor grayColor].CGColor;
layer.masksToBounds = YES;
```

效果图

![](/img/in-mpost/Core-Animation-04/border.png)

### 阴影 shadow

#### shadowOpacity

**shadowOpacity**是一个默认为0的值，必须在0.0（不可见）至1.0（完全不透明）之间。

```
layer.shadowOpacity = 1.0;
```

效果是在图层下面会有一层阴影。

![](/img/in-mpost/Core-Animation-04/shadowOpacity.png)


#### shadowOffset

**shadowOffset**是一个默认为{0，-3}的值，因为它是根据mac来的，mac坐标系与iOS相反，原本向下的阴影在iOS中就是向上的。

```
layer.shadowOpacity = 1.0;
layer.shadowOffset = CGSizeMake(0, 3);
```
效果图

![](/img/in-mpost/Core-Animation-04/shadowOffset.png)

























