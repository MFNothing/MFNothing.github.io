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