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