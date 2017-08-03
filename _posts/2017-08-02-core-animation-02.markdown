---
layout:     post
title:      "Core Animation 02的学习————寄宿图"
subtitle:   "Learning Core Animation"
date:       2017-08-02 12:00:00
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

寄宿图的像素尺寸等于视图大小乘以 contentsScale的值

效果就是当你设置的数字越大的时候，你的图片看起来会变得很小，而设置的越小的时候，你的图片看起来就会觉得放大了。

但是注意如果你设置**contentsGravity**或**contentMode**的时候用的是拉伸填满是看不出效果的。

```
layer.contentsScale = 1.0 ;
view.contentScaleFactor =  2.0;
```
效果图
![](/img/in-mpost/Core-Animation-02/Scale.png)



#### layer.maskToBounds 与 view.clipsToBounds

**maskToBounds**与**clipsToBounds**是用来决定是否显示超出边界的内容，默认为NO（即为显示）

```
layer.masksToBounds = NO;
view.clipsToBounds = YES;
```
效果图
![](/img/in-mpost/Core-Animation-02/Scale.png)


#### layer.contentsRect

**contentsRect**允许我们在图层边框里显示寄宿图的一个子域，默认为{0, 0, 1, 1} // {x, y, width, height}

```
layer.contentsRect = CGRectMake(0, 0, 0.5, 0.5);
```

效果图
![](/img/in-mpost/Core-Animation-02/contentsRect.png)


通常，多张图片可以拼合后打包整合到一张大图上一次性载入。相比多次载入不同的图片，这样做能够带来很多方面的好处：内存使用，载入时间，渲染性能等等

和bounds，frame不同，contentsRect不是按点来计算的，它使用了单位坐标，单位坐标指定在0到1之间，是一个相对值（而像素和点是绝对值）。所以它们是相对于寄宿图的尺寸的。

iOS使用了以下的坐标系统：

* **点** —— 在iOS和Mac OS中最常见的坐标体系。点就像是虚拟的像素，也被称作逻辑像素。在标准清晰度的设备上，一个点就是一个像素，但是在Retina设备上，一个点等于2*2个像素。iOS用点作为屏幕的坐标测算体系就是为了在Retina设备和普通设备上能有一致的视觉效果。
* **像素** —— 物理像素坐标并不会用来屏幕布局，但是它们在处理图片时仍然是相关的。UIImage可以识别屏幕分辨，并以点为单位指定其大小。但是一些底层的图片表示如CGImage就会使用像素，所以你要清楚在Retina设备和普通设备上，它们表现出来了不同的大小。
* **单位** —— 对于与图片大小或是图层边界相关的显示，单位坐标是一个方便的度量方式， 当大小改变的时候，也不需要再次调整。单位坐标在OpenGL这种纹理坐标系统中用得很多，Core Animation中也用到了单位坐标。

#### layer.contentsCenter 与 view.contentStretch

**contentsCenter**与**contentStretch**用于设置可以拉伸的区域默认为{0, 0, 1, 1}，设置的区域即为被拉伸的区域，用的地方比如进度条的图片 // {x, y, width, height}

```
layer.contentsCenter = CGRectMake(0.26, 0.3, 0.45, 0.4);
view.contentStretch = CGRectMake(0.26, 0.3, 0.45, 0.4);// 被废弃了，所以最好不用,但是在Interface Builder中可以设置这个属性
```
效果图

![](/img/in-mpost/Core-Animation-02/contentsCenter.png)

#### 用Core Graphics直接绘制
CALayer有一个可选的delegate属性，实现了CALayerDelegate协议，当CALayer需要一个内容特定的信息时，就会从协议中请求。你只需要调用你想调用的方法，CALayer会帮你做剩下的。（delegate属性被声明为id类型，所有的代理方法都是可选的。

当需要被重绘时，CALayer会请求它的代理给它一个寄宿图来显示。它通过调用下面这个方法做到的:

```
- (void)displayLayer:(CALayer *)layer;
```

趁着这个机会，如果代理想直接设置contents属性的话，它就可以这么做，不然没有别的方法可以调用了。如果代理不实现-displayLayer:方法，CALayer就会转而尝试调用下面这个方法：

```
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;
```
在调用这个方法之前，CALayer创建了一个合适尺寸的空寄宿图（尺寸由bounds和contentsScale决定）和一个Core Graphics的绘制上下文环境，为绘制寄宿图做准备，它作为ctx参数传入。

```
#import "ViewController.h"
#import <QuartzCore/QuartzCore.h>
@interface ViewController () <CALayerDelegate>
@property (nonatomic, strong) UIView *baseView;
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    UIImage *demoImage = [UIImage imageNamed:@"demo"];
    self.baseView = [[UIView alloc] initWithFrame:CGRectMake(50, 300, 100, 100)];
    [self.baseView setBackgroundColor:[[UIColor yellowColor] colorWithAlphaComponent:0.3]];
    self.baseView.layer.contents = (__bridge id)demoImage.CGImage;
    [self.view addSubview:self.baseView];
    //create sublayer
    CALayer *blueLayer = [CALayer layer];
    blueLayer.frame = CGRectMake(0, 0, 100.0f, 100.0f);
    blueLayer.delegate = self;
    blueLayer.contentsScale = [UIScreen mainScreen].scale;
    [self.baseView.layer addSublayer:blueLayer];
    [blueLayer display];
}
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx
{
    //draw a thick black circle
    CGContextSetLineWidth(ctx, 5.0f);
    CGContextSetStrokeColorWithColor(ctx, [UIColor blackColor].CGColor);
    CGContextStrokeEllipseInRect(ctx, layer.bounds);
}
```
效果图
![](/img/in-mpost/Core-Animation-02/drawLayer.png)

注意一下一些有趣的事情：

* 我们在blueLayer上显式地调用了-display。不同于UIView，当图层显示在屏幕上时，CALayer不会自动重绘它的内容。它把重绘的决定权交给了开发者。
* 尽管我们没有用masksToBounds属性，绘制的那个圆仍然沿边界被裁剪了。这是因为当你使用CALayerDelegate绘制寄宿图的时候，并没有对超出边界外的内容提供绘制支持。

现在你理解了CALayerDelegate，并知道怎么使用它。但是除非你创建了一个单独的图层，你几乎没有机会用到CALayerDelegate协议。因为当UIView创建了它的宿主图层时，它就会自动地把图层的delegate设置为它自己，并提供了一个-displayLayer:的实现，那所有的问题就都没了。

当使用寄宿了视图的图层的时候，你也不必实现-displayLayer:和-drawLayer:inContext:方法来绘制你的寄宿图。通常做法是实现UIView的-drawRect:方法，UIView就会帮你做完剩下的工作，包括在需要重绘的时候调用-display方法。

