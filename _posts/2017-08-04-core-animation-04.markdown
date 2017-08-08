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

#### shadowRadius

**shadowRadius**属性控制着阴影的模糊度，当它的值是0的时候，阴影就和视图一样有一个非常确定的边界线。当它的值越大时，会变得很淡，也会更宽，后面会发现看不见了。

```
baseView1.layer.shadowRadius = 20;
```

效果图
![](/img/in-mpost/Core-Animation-04/shadowRadius.png)

### 阴影剪裁

和图层边框不同，图层的阴影来源于其内容的确切形状，而不是仅仅是边界和cornerRadius。为了计算出阴影的形状，Core Animation会将寄宿图（包括子视图，如果有的话）考虑在内，然后通过这些来完美搭配图层形状从而创建一个阴影。

只含寄宿图的效果图

![](/img/in-mpost/Core-Animation-04/shadowPic.png)

当我们使用**masksToBounds**属性的时候，阴影部分也会被剪裁掉（如果阴影部分超出显示范围）。

所以可以做的是利用两个视图，一个视图包含另一个，大小相同，外部视图使用阴影，而内部视图进行剪切。（即多用一个视图来展示阴影）

### shadowPath属性

实时计算阴影也是非常消耗资源的，尤其是当图层有多个子图层，每个图层还有一个有透明效果的寄宿图的时候。可以通过指定一个shadowPath来提高性能。

**shadowPath**是一个CGPathRef类型（一个指向CGPath的指针）。CGPath是一个Core Graphics对象，用来指定任意的一个矢量图形。我们可以通过这个属性独立于图层形状之外指定阴影的形状。


```
layer.shadowOpacity = 0.5;
CGMutablePathRef squarePath = CGPathCreateMutable();
CGPathAddRect(squarePath, NULL, baseView.bounds);
layer.shadowPath = squarePath;
CGPathRelease(squarePath);
```

效果图
![](/img/in-mpost/Core-Animation-04/shadowPath.png)

### 图层蒙版 mask

CALayer有一个属性叫做mask，mask图层定义了父图层的部分可见区域。

mask属性就像是一个饼干切割机，mask图层实心的部分会被保留下来，其他的则会被抛弃。

这个属性本身就是个CALayer类型，有和其他图层一样的绘制和布局属性。

```
UIImage *image = [UIImage imageNamed:@"demo"];
CALayer *maskLayer = [CALayer layer];
maskLayer.frame = CGRectMake(0, 0, 100, 100);
maskLayer.contents = (__bridge id)image.CGImage;
UIView *view = [[UIView alloc]initWithFrame:CGRectMake(200, 300, 100, 100)];
[view setBackgroundColor:[[UIColor redColor] colorWithAlphaComponent:0.3]];
view.layer.mask = maskLayer;
```
效果图

![](/img/in-mpost/Core-Animation-04/mask.png)


### 拉伸过滤 minificationFilter和magnificationFilter


**minificationFilter**和**magnificationFilter**属性分别是设置图片拉伸过滤的方法。

CALayer有三种拉伸过滤的方法：

* kCAFilterLinear
* kCAFilterNearest
* kCAFilterTrilinear

默认都为kCAFilterLinear。

**kCAFilterLinear**这个过滤器采用双线性滤波算法，它在大多数情况下都表现良好。双线性滤波算法通过对多个像素取样最终生成新的值，得到一个平滑的表现不错的拉伸。但是当放大倍数比较大的时候图片就模糊不清了。

**kCAFilterTrilinear**和kCAFilterLinear非常相似，大部分情况下二者都看不出来有什么差别。但是，较双线性滤波算法而言，三线性滤波算法存储了多个大小情况下的图片（也叫多重贴图），并三维取样，同时结合大图和小图的存储进而得到最后的结果。


**kCAFilterNearest**是一种比较武断的方法。从名字不难看出，这个算法（也叫最近过滤）就是取样最近的单像素点而不管其他的颜色。这样做非常快，也不会使图片模糊。但是，最明显的效果就是，会使得压缩图片更糟，图片放大之后也显得块状或是马赛克严重。

![](/img/in-mpost/Core-Animation-04/kCAFilter.png)

#### LED钟表demo





