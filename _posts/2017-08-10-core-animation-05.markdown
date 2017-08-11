---
layout:     post
title:      "Core Animation 05的学习————变换"
subtitle:   "Learning Core Animation"
date:       2017-08-10 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Core Animation
---


## 变换

### 仿射变换（AffineTransform）

**仿射变换**简单来说是平移、缩放、旋转、翻转和错切这五种变换的组合。

**错切**实际上是平面景物在投影平面上的非垂直投影。

#### 简单变化（一次性的）

**Core Graphics**提供一些简单的函数让我们能做简单变换。

注意一下的操作都是改变的视图坐标系,并且是以初始坐标为基准

-----

**旋转** CGAffineTransformMakeRotation(CGFloat angle) 

注意这里是以初始位置为基准，将坐标系顺时针旋转了
 
```
// 顺时针旋转45度
CGAffineTransform transform = CGAffineTransformMakeRotation(M_PI_4);
view.layer.affineTransform = transform;
```
效果图
![](/img/in-mpost/Core-Animation-05/Rotation.png)

**缩放**  CGAffineTransformMakeScale(CGFloat sx, CGFloat sy)

以初始位置为基准,在x轴方向上缩放x倍,在y轴方向上缩放y倍

需要注意的是当值sx为负数的时候，会将坐标系以Y为轴翻转过来，当sy为负数的时候，会将坐标系以X为轴翻转过来。

```
// 整体缩小一半并X方向翻转
CGAffineTransform transform = CGAffineTransformMakeScale(1, 0.5);
view.layer.affineTransform = transform;
```
效果图
![](/img/in-mpost/Core-Animation-05/Scale.png)

**平移** CGAffineTransformMakeTranslation(CGFloat tx, CGFloat ty)

以初始位置为基准,在x轴方向上平移x单位,在y轴方向上平移y单位

```
// 向右平移50个点
CGAffineTransform transform = CGAffineTransformMakeTranslation(50, 0);
view.layer.affineTransform = transform;
```
效果图
![](/img/in-mpost/Core-Animation-05/Translation.png)

#### 混合变换

混合变换可以下面的函数实现，将多个操作叠加在一起，不过是以上一个改变了的坐标系为基础。

* CGAffineTransformRotate(CGAffineTransform t, CGFloat angle)     
* CGAffineTransformScale(CGAffineTransform t, CGFloat sx, CGFloat sy)      
* CGAffineTransformTranslate(CGAffineTransform t, CGFloat tx, CGFloat ty)

下面是先缩放{-0.5，0.5}，再旋转M_PI_4，最后平移50个点

```
	UIImage *image = [UIImage imageNamed:@"demo"];
	CGAffineTransform transIndentity = CGAffineTransformIdentity;
	for (int i = 0; i < 3; i++) {
		UIView *baseView = [[UIView alloc]initWithFrame:CGRectMake(50, 300, 100, 100)];
		baseView.layer.contents = (__bridge id)image.CGImage;
		[self.view addSubview:baseView];
		if (i == 0) {
			baseView.backgroundColor = [[UIColor orangeColor] colorWithAlphaComponent:0.3];
			transIndentity = CGAffineTransformScale(transIndentity, -0.5, 0.5);
		}else if(i == 1) {
			baseView.backgroundColor = [[UIColor redColor] colorWithAlphaComponent:0.3];
			transIndentity = CGAffineTransformRotate(transIndentity, M_PI_4);
		}else {
			baseView.backgroundColor = [[UIColor yellowColor] colorWithAlphaComponent:0.3];
			transIndentity = CGAffineTransformTranslate(transIndentity, 50, 0);
		}
		baseView.layer.affineTransform = transIndentity;
	}
```

效果图
![](/img/in-mpost/Core-Animation-05/mixTransform.png)


#### 错切变换

直接设置CGAffineTransform的值，以达到错切的效果。

但是一般用不上，但是可以知道的是上面变换的函数也是通过改变这些值来进行变换的。

```
struct CGAffineTransform {
  CGFloat a, b, c, d;
  CGFloat tx, ty;
};
```

如果想具体了解这一转换过程，可以去搜索一下这个结构体。

```
CGAffineTransform transIndentity = CGAffineTransformIdentity;
transIndentity.c = -1;
transIndentity.b = 0;
view.layer.affineTransform = transIndentity;
```
效果图
![](/img/in-mpost/Core-Animation-05/shear.png)

### 3D变换

就是X，Y坐标系的基础上加上了一个垂直于屏幕的Z坐标。

对于旋转来说默认正数都是顺时针方向的。

与上面对应的操作的函数：

* CATransform3DMakeRotation(CGFloat angle, CGFloat x, CGFloat y, CGFloat z)
* CATransform3DMakeScale(CGFloat sx, CGFloat sy, CGFloat sz) 
* CATransform3DMakeTranslation(Gloat tx, CGFloat ty, CGFloat tz)

------

**旋转** 都是以最初的坐标系来旋转的（可以自己拿张纸来模拟）

显示出来效果是垂直投影下去的样子。

```
CATransform3D transform = CATransform3DMakeRotation(M_PI_4, 1, 1, 1) ;
view.layer.transform = transform;
```
效果图
![](/img/in-mpost/Core-Animation-05/ThreeDRotation.png)


#### 透视投影

```
struct CATransform3D
{
  CGFloat m11, m12, m13, m14;
  CGFloat m21, m22, m23, m24;
  CGFloat m31, m32, m33, m34;
  CGFloat m41, m42, m43, m44;
};
typedef struct CATransform3D CATransform3D;
```

在这个结构体中，m34是用来做透视的。

m34的默认值是0，我们可以通过设置m34为-1.0 / d来应用透视效果，d代表了想象中视角相机和屏幕之间的距离，以像素为单位，那应该如何计算这个距离呢？实际上并不需要，大概估算一个就好了。

视角相机的位置会在下面介绍，一般位于视图的中央。

#### 灭点

**灭点**：指的是立体图形各条边的延伸线所产生的相交点。

当立方体离我们越来越远的时候，它就会不断变小，直到跟灭点一样看不见。（参照透视图）

Core Animation定义了这个点位于变换图层的anchorPoint这就是说，当图层发生变换时，这个点永远位于图层变换之前anchorPoint的位置。

改变position，也会使灭点改变，做3D变换的时候要时刻记住这一点，当你视图通过调整m34来让它更加有3D效果，应该首先把它放置于屏幕中央，然后通过平移来把它移动到指定位置（而不是直接改变它的position），这样所有的3D图层都共享一个灭点。

#### sublayerTransform

这个是设置所有子视图透视的m34值的。这个属性可以确保在变换之前都在屏幕中央共享同一个position。

```
CATransform3D perspective = CATransform3DIdentity;
perspective.m34 = - 1.0 / 500.0;
view.layer.sublayerTransform = perspective;
rightView.layer.transform = CATransform3DMakeRotation(-M_PI_4, 0, 1, 0);
leftView.layer.transform = CATransform3DMakeRotation(M_PI_4, 0, 1, 0);
```
效果图
![](/img/in-mpost/Core-Animation-05/sublayerTransform.png)


#### 背面

通过以Y为轴3D旋转180度，我们可以看见视图的背面，与正面是一个镜面的效果。

在Layer中我们可以通过设置doubleSided来控制是否绘制背面，默认为YES。设置为NO后，翻面过去什么都没有显示了。


#### 扁平化图层

在二维不存在扁平化，本来就是扁的。但是在三维中，绘制在屏幕的视图其实是一个平面，只是看起来像在做3D变换。

比如下面这个例子，父视图以Y轴旋转45度，子视图想通过反向旋转抵消父视图的旋转效果。

```
	// 设置透视点
	CATransform3D subTransform = CATransform3DIdentity;
	subTransform.m34 = -1.0 / 500;
	self.view.layer.sublayerTransform = subTransform;
	// 创建效果视图
	CGFloat width = CGRectGetWidth(self.view.frame) / 2;
	UIView *superView = [[UIView alloc]initWithFrame:CGRectMake(width + 50, 300, 100, 100)];
	superView.backgroundColor = [[UIColor grayColor] colorWithAlphaComponent:0.5];
	UIView *subView = [[UIView alloc]initWithFrame:CGRectMake(25, 25, 50, 50)];
	subView.backgroundColor = [[UIColor yellowColor] colorWithAlphaComponent:0.5];
	[superView addSubview:subView];
	[self.view addSubview:superView];
    superView.layer.transform = CATransform3DMakeRotation(M_PI_4, 0, 1, 0);
	subView.layer.transform = CATransform3DMakeRotation(-M_PI_4, 0, 1, 0);
	// 对比视图
	UIView *superView1 = [[UIView alloc]initWithFrame:CGRectMake(width - 150, 300, 100, 100)];
	superView1.backgroundColor = [[UIColor grayColor] colorWithAlphaComponent:0.5];
	UIView *subView1 = [[UIView alloc]initWithFrame:CGRectMake(25, 25, 50, 50)];
	subView1.backgroundColor = [[UIColor yellowColor] colorWithAlphaComponent:0.5];
	[superView1 addSubview:subView1];
	[self.view addSubview:superView1];
```

效果图
![](/img/in-mpost/Core-Animation-05/flattening.png)

旋转的效果并没有被抵消，是因为视图在屏幕中已经扁平化了，子视图旋转的时候，已经不是原来的大小了，虽然它旋转了回去。

所有场景里面绘制的东西并不会随着你观察它的角度改变而发生变化；图层也是同样的道理。

这使得用Core Animation创建非常复杂的3D场景变得十分困难。你不能够使用图层树去创建一个3D结构的层级关系--在相同场景下的任何3D表面必须和同样的图层保持一致，这是因为每个的父视图都把它的子视图扁平化了。


### 固体对象

```
@interface ViewController ()
@property (nonatomic, strong) NSMutableArray *viewArrs;
@end
@implementation ViewController
- (void)viewDidLoad {
	[super viewDidLoad];
	// 初始化UI
	[self initUI];
	// 设置透视点
	CATransform3D perspective = CATransform3DIdentity;
	perspective.m34 = -1.0 / 500;
	perspective = CATransform3DRotate(perspective, -M_PI_4, 1, 0, 0);
	perspective = CATransform3DRotate(perspective, -M_PI_4, 0, 1, 0);
	self.view.layer.sublayerTransform = perspective;
	// 设置视图 1
	CATransform3D transform = CATransform3DMakeTranslation(0, 0, 50);
	[self setTransform:transform atIndex:0];
	// 设置视图 2
	transform = CATransform3DMakeTranslation(50, 0, 0);
	transform = CATransform3DRotate(transform, M_PI_2, 0, 1, 0);
	[self setTransform:transform atIndex:1];
	// 设置视图 3
	transform = CATransform3DMakeTranslation(0, -50, 0);
	transform = CATransform3DRotate(transform, M_PI_2, 1, 0, 0);
	[self setTransform:transform atIndex:2];
	// 设置视图 4
	transform = CATransform3DMakeTranslation(0, 50, 0);
	transform = CATransform3DRotate(transform, -M_PI_2, 1, 0, 0);
	[self setTransform:transform atIndex:3];
	// 设置视图 5
	transform = CATransform3DMakeTranslation(-50, 0, 0);
	transform = CATransform3DRotate(transform, -M_PI_2, 0, 1, 0);
	[self setTransform:transform atIndex:4];
	// 设置视图 6
	transform = CATransform3DMakeTranslation(0, 0, -50);
	transform = CATransform3DRotate(transform, M_PI, 0, 1, 0);
	[self setTransform:transform atIndex:5];
}
- (void)setTransform:(CATransform3D)transform atIndex:(NSInteger)index
{
	UIView *view = self.viewArrs[index];
	view.layer.transform = transform;
}
- (void)initUI
{
	self.viewArrs = [NSMutableArray array];
	CGFloat width = CGRectGetWidth(self.view.frame) / 2;
	CGFloat height = CGRectGetHeight(self.view.frame) / 2;
	for (int i = 0; i < 6; i++) {
		UIView *view = [[UIView alloc] initWithFrame:CGRectMake(width - 50, height - 50, 100, 100)];
		view.layer.borderColor = [UIColor blackColor].CGColor;
		view.layer.borderWidth = 5.0f;
		view.backgroundColor = [[UIColor yellowColor]colorWithAlphaComponent:0.8];
		UIButton *numberButton = [[UIButton alloc] initWithFrame:CGRectMake(0, 0, 100, 100)];
		[numberButton addTarget:self action:@selector(buttonClick:) forControlEvents:UIControlEventTouchUpInside];
		numberButton.tag = i;
		UILabel *numberLabel = [[UILabel alloc] initWithFrame:CGRectMake(25, 25, 50, 50)];
		numberLabel.textColor = [UIColor blackColor];
		numberLabel.font = [UIFont systemFontOfSize:20];
		numberLabel.textAlignment = NSTextAlignmentCenter;
		numberLabel.text = [NSString stringWithFormat:@"%d", i + 1];
		[view addSubview:numberButton];
		[view addSubview:numberLabel];
		[self.viewArrs addObject:view];
		[self.view addSubview:view];
	}
}
- (void)buttonClick:(id)sender
{
	UIButton *button = (UIButton *)sender;
	NSLog(@"button %ld", button.tag + 1);
}
@end
```


效果图

![](/img/in-mpost/Core-Animation-05/fixModel.png)

### 点击事件

点击事件的处理是由视图在父视图的顺序决定的，并不是3D空间中的Z轴顺序。所以当我们想点击3时，会被4、5、6拦截(取决于点击位置)，如果4、5、6有方法可以响应的话。事件响应并不会因为变换的操作而发生改变。






















