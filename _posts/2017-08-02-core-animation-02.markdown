---
layout:     post
title:      "Core Animation 02的学习————寄宿图"
subtitle:   "Learning Core Animation"
date:       2017-08-2 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Core Animation
---

## 寄宿图

### 什么是寄宿图（图层中包含的图 contents）

**寄宿图**是CALayer中一个叫**contents**的属性。

**属性类型**为**id**（但是在iOS中如果传入的不是CGImage，那么得到的图层会是空白的），之所以为id类型是因为在Mac OS系统上，这个属性传入CGImage和NSImage类型数据都是作用的。

但是直接给contents这个属性赋值UIImage中的CGImage属性会有问题（在ARC中），因为它返回的是一个CGImageRef，这不一个真正的Cocoa对象，而是一个Core Foundation类型。所以应该转换类型（在ARC中，在MRC中不用）

```
layer.contents = (__bridge id)image.CGImage;
```

### contens的相关属性介绍

#### layer.contentsGravity 与 view.contentMode

**contentsGravity**和**contentMode**目的是为了决定图层在边界中怎么对齐（默认都为拉伸填充满，与AspectFill的区别在于，AspectFill不会把图片拉变形，超出就超出，与其他对齐方式（如Center）的区别在于Center不会去拉扯图片，它有好大就好大）

|对齐方式|contentsGravity|contentMode|
|:----:|:----:|:----:|
|居中|kCAGravityCenter|UIViewContentModeCenter|
|居于顶部|kCAGravityTop|UIViewContentModeTop|
|居于底部|kCAGravityBottom|UIViewContentModeBottom|
|居左|kCAGravityLeft|UIViewContentModeLeft|
|居右|kCAGravityRight|UIViewContentModeRight|
|居于左上|kCAGravityTopLeft|UIViewContentModeTopLeft|
|居于右上|kCAGravityTopRight|UIViewContentModeTopRight|
|居于左下|kCAGravityBottomLeft|UIViewContentModeBottomLeft|
|居于右下|kCAGravityBottomRight|UIViewContentModeBottomRight|
|拉伸填满|kCAGravityResize|UIViewContentModeScaleToFill|
|保持形状不变，拉伸使其刚好能展示图片，有地方为空都没关系|kCAGravityResizeAspect|UIViewContentModeScaleAspectFit|
|保持形状不变，拉伸填充满,超出也没有关系|kCAGravityResizeAspectFill|UIViewContentModeScaleAspectFill|

区别是**contentsGravity**传入的是NSString(输入上面的kCAGravityCenter就可以的不用输入@“kCAGravityCenter”,如果你强行输入这个也行)，而**contentMode**用的是枚举

```
layer.contentsGravity = kCAGravityResizeAspectFill;
view.contentMode = UIViewContentModeCenter;
```
效果图
![](/img/in-mpost/Core-Animation-02/Center-AspectFill.png)

```
layer.contentsGravity = kCAGravityResize;
view.contentMode = UIViewContentModeScaleAspectFill;
```
效果图
![](/img/in-mpost/Core-Animation-02/Resize-AspectFill.png)

#### layer.contensScale 与 view.contentScaleFactor

**contensScale**和**contentScaleFactor**的目的是设置图片一个点用几个像素图绘制，设置为1.0，将会以每个点1个像素绘制图片，如果设置为2.0，则会以每个点2个像素绘制图片，这就是我们熟知的Retina屏幕。

效果就是当你设置的数字越大的时候，你的图片看起来会变得很小，而设置的越小的时候，你的图片看起来就会觉得放大了。

但是注意如果你设置**contentsGravity**或**contentMode**的时候用的是拉伸填满是看不出效果的。

```
layer.contentsScale = 1.0 ;
view.contentScaleFactor =  2.0;
```
效果图
![](/img/in-mpost/Core-Animation-02/Scale.png)

```

- (void)viewDidLoad {
    [super viewDidLoad];
    UIImage *demoImage = [UIImage imageNamed:@"demo"];
    // topView 使用图层方法
    UIView *topView = [[UIView alloc] initWithFrame:CGRectMake(50, 50, 100, 50)];
    [topView setBackgroundColor:[[UIColor yellowColor] colorWithAlphaComponent:0.3]];
    topView.layer.contents = (__bridge id)demoImage.CGImage;
    // contentsGravity 默认为resize，也改变大小就是让它能放入这个view中
   // topView.layer.contentsGravity = kCAGravityResizeAspect;
    topView.layer.contentsGravity = kCAGravityCenter;
    topView.layer.contentsScale = demoImage.scale;
    topView.layer.masksToBounds = YES;
    // bottomView 使用UIImageView的方法
    UIImageView *bottomView = [[UIImageView alloc] initWithFrame:CGRectMake(50, 300, 100, 50)];
    [bottomView setBackgroundColor:[[UIColor yellowColor] colorWithAlphaComponent:0.3]];
    [bottomView setImage:demoImage];
    // contentMode 默认是fill填充满
   // bottomView.contentMode = UIViewContentModeScaleAspectFit;
    bottomView.contentMode = UIViewContentModeCenter;
    bottomView.contentScaleFactor = 1.0;
    bottomView.clipsToBounds = YES;
    
    [self.view addSubview:topView];
    [self.view addSubview:bottomView];
}

```