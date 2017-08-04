---
layout:     post
title:      "Core Animation 03的学习————图层几何学"
subtitle:   "Learning Core Animation"
date:       2017-08-03 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Core Animation
---

## 图层几何学

### 三个重要布局属性

UIView中叫 frame，bounds和center

CALayer中叫frame，bounds和position

**frame**代表对于父图层来说你在的位置和大小

**bounds**代表对于自己本身来说你的位置和大小({0，0}通常是图层的左上角)

**positon**和**center**代表对于父图层来说它的anchorPoint（旋转或缩放的基准点）所在的位置

对于视图或者图层来说，frame并不是一个非常清晰的属性，它其实是一个虚拟属性，是根据bounds，position和transform计算而来，所以当其中任何一个值发生改变，frame都会变化。相反，改变frame的值同样会影响到他们当中的值。

如旋转一个图片之后，frame会变化。

### 锚点 anchorPoint

anchorPoint用单位坐标来描述，也就是图层的相对坐标，图层左上角是{0, 0}，右下角是{1, 1}，因此默认坐标是{0.5, 0.5}。anchorPoint可以通过指定x和y值小于0或者大于1，使它放置在图层范围之外。

注意你改变anchorPoint的值并不会改变position或center的位置,所以frame会变。

比如一张图，你把它的anchorPoint设置为了{0，0},这时他旋转的点就变到了左上角，而对于父图层来说，它的anchorPoint并没有变化也就是position没有变化，所以只有改变图层在父图层的位置作调整，让图层的左上角移动到position的位置，所以看起来图层向右下方移动了。

```
layer.anchorPoint = CGPointMake(0, 0);
```

效果图

![](/img/in-mpost/Core-Animation-03/anchorPoint.png)


如果只是想要改变锚点但是不想让它的frame的改变,在设置anchorPoint之前记录一下之前的frame中需要的值

```
CGFloat minX = CGRectGetMinX(view.frame);
CGFloat minY = CGRectGetMinY(view.frame);
view.layer.anchorPoint = CGPointMake(0, 0);
view.center = CGPointMake(minX, minY);
// 或者
view.layer.position = CGPointMake(minX, minY);
```

### 坐标系

UIView和CALayer都提供一些方法来判断点或者视图（图层）在某个视图（图层）坐标系下的相对坐标或frame值


```
// UIView
// 返回调用者中的point这个点在view这个坐标系中的相对位置
- (CGPoint)convertPoint:(CGPoint)point toView:(nullable UIView *)view;
// 返回view中在point这个点在调用者这个坐标系中的位置
- (CGPoint)convertPoint:(CGPoint)point fromView:(nullable UIView *)view;
// 返回调用者中的rect在view这个坐标系中的相对frame
- (CGRect)convertRect:(CGRect)rect toView:(nullable UIView *)view;
// 返回view中rect在调用者这个坐标系中的相对frame
- (CGRect)convertRect:(CGRect)rect fromView:(nullable UIView *)view;
// CALayer
// 返回调用者中的point这个点在layer这个坐标系中的相对位置
- (CGPoint)convertPoint:(CGPoint)point fromLayer:(CALayer *)layer;
// 返回layer中在point这个点在调用者这个坐标系中的位置 
- (CGPoint)convertPoint:(CGPoint)point toLayer:(CALayer *)layer; 
// 返回调用者中的rect在layer这个坐标系中的相对frame
- (CGRect)convertRect:(CGRect)rect fromLayer:(CALayer *)layer;
// 返回layer中rect在调用者这个坐标系中的相对frame
- (CGRect)convertRect:(CGRect)rect toLayer:(CALayer *)layer;
```

例如，一个view的frame的为{50，100，100，100}名为baseView，然后调用

```
[self.view convertPoint:CGPointMake(200, 300) toView:self.baseView] // self.view为baseView的父视图
```

返回的是{150，200}，然后调用

```
[self.view convertPoint:CGPointMake(200, 300) toView:self.baseView] // self.view为baseView的父视图
```

返回的是{250，400}

#### 翻转的几何结构

常规说来，在iOS上，一个图层的position位于父图层的左上角，但是在Mac OS上，通常是位于左下角。但是我们可以通过设置layer的geometryFlipped属性，使其变为左下，默认为NO。当其设置为YES之后，在iOS中的这个图层的子图层或子视图都将以左下角作为坐标轴的起点。


```
UIImage *image = [UIImage imageNamed:@"demo"];
CALayer *baseLayer = [[CALayer alloc]init];
baseLayer.frame = CGRectMake(50, 50, 100, 100);
baseLayer.backgroundColor = [[UIColor yellowColor]colorWithAlphaComponent:0.3].CGColor;
baseLayer.contents =(__bridge id)image.CGImage;
[self.view.layer addSublayer:baseLayer];
CGFloat maxY = CGRectGetMaxY(self.view.frame);
UIView *baseView = [[UIView alloc]initWithFrame:CGRectMake(50, maxY - 150, 100, 100)];
[baseView setBackgroundColor:[[UIColor redColor] colorWithAlphaComponent:0.3]];
baseView.layer.contents = (__bridge id)image.CGImage;
[self.view addSubview:baseView];
self.view.layer.geometryFlipped = YES;
```
效果图
![](/img/in-mpost/Core-Animation-03/geometryFlipped.png)


#### Z坐标系

图层是一层一层的，所以Z代表他的厚度，一般都是0，但是通过设置值后可以让下面的图层移动到上面去。

```
layer.zPosition = 1.0f;
```

#### Hit Testing

CALayer对响应链一无所知，所以它不能直接处理触摸事件或者手势。但是它有一系列的方法帮你处理事件：-containsPoint:和-hitTest:


-containsPoint:接受一个在本图层坐标系下的CGPoint，如果这个点在图层frame范围内就返回YES。

注意这个需要把点通过convert转换到本坐标系下再判断

```
- (BOOL)containsPoint:(CGPoint)p;
```

-hitTest:方法同样接受一个CGPoint类型参数，它返回图层本身，或者包含这个坐标点的叶子节点图层，而不是BOOL类型。如果点在最外面的图层外，则返回nil。

```
- (nullable CALayer *)hitTest:(CGPoint)p;
```
注意当调用图层的-hitTest:方法时，测算的顺序严格依赖于图层树当中的图层顺序（和UIView处理事件类似）。之前提到的zPosition属性可以明显改变屏幕上图层的顺序，但不能改变触摸事件被处理的顺序。

这意味着如果改变了图层的z轴顺序，你会发现将不能够检测到最前方的视图点击事件，这是因为被另一个图层遮盖住了，虽然它的zPosition值较小，但是在图层树中的顺序靠前。我们将在第五章详细讨论这个问题。

#### 自动布局

当使用视图的时候，可以充分利用UIView类接口暴露出来的UIViewAutoresizingMask和NSLayoutConstraintAPI，但如果想随意控制CALayer的布局，就需要手工操作。最简单的方法就是使用CALayerDelegat（注意要设置代理，并且不要用控制器的layer设置代理）如下函数：

- (void)layoutSublayersOfLayer:(CALayer *)layer;
当图层的bounds发生改变，或者图层的-setNeedsLayout方法被调用的时候，这个函数将会被执行。这使得你可以手动地重新摆放或者重新调整子图层的大小，但是不能像UIView的autoresizingMask和constraints属性做到自适应屏幕旋转。

这也是为什么最好使用视图而不是单独的图层来构建应用程序的另一个重要原因之一。



### 时钟demo

效果图

![](/img/in-mpost/Core-Animation-03/clock.png)

```
#import "ViewController.h"
#import <QuartzCore/QuartzCore.h>
@interface ViewController ()
@property (nonatomic, strong) UIView *baseView;
@property (nonatomic, strong) UIImageView *hourImageView;
@property (nonatomic, strong) UIImageView *minuteImageView;
@property (nonatomic, strong) UIImageView *secondImageView;
@property (nonatomic, weak) NSTimer *timer;
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    _hourImageView = [[UIImageView alloc]initWithImage:[UIImage imageNamed:@"hour_hand"]];
    _minuteImageView = [[UIImageView alloc]initWithImage:[UIImage imageNamed:@"minute_hand"]];
    _secondImageView = [[UIImageView alloc]initWithImage:[UIImage imageNamed:@"second_hand"]];
    _hourImageView.layer.anchorPoint = CGPointMake(0.5 , 0.9);
    _minuteImageView.layer.anchorPoint = CGPointMake(0.5 , 0.9);
    _secondImageView.layer.anchorPoint = CGPointMake(0.5 , 0.9);
    UIImage *image = [UIImage imageNamed:@"demo"];
    self.baseView = [[UIView alloc]initWithFrame:CGRectMake(50, 300, 100, 100)];
    [self.baseView setBackgroundColor:[[UIColor yellowColor] colorWithAlphaComponent:0.3]];
    self.baseView.layer.contents = (__bridge id)image.CGImage;
    CGFloat centerX = CGRectGetWidth(self.baseView.frame) / 2;
    CGFloat centerY = CGRectGetHeight(self.baseView.frame) / 2;
    _hourImageView.center = CGPointMake(centerX, centerY);
    _minuteImageView.center = CGPointMake(centerX, centerY);
    _secondImageView.center = CGPointMake(centerX, centerY);
    [self.baseView addSubview:_hourImageView];
    [self.baseView addSubview:_minuteImageView];
    [self.baseView addSubview:_secondImageView];
    [self.view addSubview:self.baseView];
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(tick) userInfo:nil repeats:YES];
}
- (void)tick
{
    //convert time to hours, minutes and seconds
    NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSCalendarIdentifierGregorian];
    NSUInteger units = NSCalendarUnitHour | NSCalendarUnitMinute | NSCalendarUnitSecond;
    NSDateComponents *components = [calendar components:units fromDate:[NSDate date]];
    CGFloat hoursAngle = (components.hour / 12.0) * M_PI * 2.0;
    //calculate hour hand angle //calculate minute hand angle
    CGFloat minsAngle = (components.minute / 60.0) * M_PI * 2.0;
    //calculate second hand angle
    CGFloat secsAngle = (components.second / 60.0) * M_PI * 2.0;
    //rotate hands
    self.hourImageView.transform = CGAffineTransformMakeRotation(hoursAngle);
    self.minuteImageView.transform = CGAffineTransformMakeRotation(minsAngle);
    self.secondImageView.transform = CGAffineTransformMakeRotation(secsAngle);
}
```