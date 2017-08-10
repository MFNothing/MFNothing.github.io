---
layout:     post
title:      "Core Animation 04的学习————变换"
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
![](/img/in-mpost/Core-Animation-05/3DRatation.png)






