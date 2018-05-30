---
layout:     post
title:      "控制器view加载原理学习笔记"
subtitle:   "控制器view加载原理学习笔记"
date:       2018-05-29 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Application
---

## 控制器view加载原理

### 几种与视图相关创建控制器的方式

**创建跟控制器相同名称的xib文件**

![](/img/in-mpost/Controller-Loading-Principle/CreateController1.gif)

```
FirstViewController *vc = [[FirstViewController alloc] init];
```

**先创建一个控制器，然后将一个xib的view与控制器的view关联**

![](/img/in-mpost/Controller-Loading-Principle/CreateController2.gif)

这个方式跟第一个创建方式差不多，只是第一种方式默认做了第二种关联 view 的操作。

```
SecondViewController *vc = [[SecondViewController alloc] init];
```

**先创建一个控制器，然后将一个xib中的控制器与之关联**

![](/img/in-mpost/Controller-Loading-Principle/CreateController3.gif)

```
ThirdViewController *vc = [[[NSBundle mainBundle] loadNibNamed: @"ThirdViewController" owner: nil options: nil] lastObject];
```

**先创建一个控制器，然后将storyboard中的一个控制器与之关联**

![](/img/in-mpost/Controller-Loading-Principle/CreateController4.gif)

```
FourthViewController *vc = [[UIStoryboard storyboardWithName: @"Main" bundle: nil] instantiateViewControllerWithIdentifier: @"FourthViewController"];
```


### 定义一个控制器

```
#import <UIKit/UIKit.h>

@interface FirstViewController : UIViewController

@end

#import "FirstViewController.h"

@interface FirstViewController ()

@end

@implementation FirstViewController

//- (void)loadView
//{
//    [super loadView];
//    NSLog(@"%s", __func__);
//}

- (instancetype)initWithCoder:(NSCoder *)aDecoder
{
    NSLog(@"%s", __func__);
    if (self= [super initWithCoder:aDecoder]) {
        
    }
    return self;
}

- (void)viewDidLayoutSubviews
{
    NSLog(@"%s", __func__);
    [super viewDidLayoutSubviews];
}

- (void)viewWillLayoutSubviews
{
    NSLog(@"%s", __func__);
    [super viewWillLayoutSubviews];
}

- (instancetype)init
{
    NSLog(@"%s", __func__);
    if (self = [super init]) {
        
    }
    return self;
}

- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
    NSLog(@"%s", __func__);
    if (self = [super initWithNibName: nibNameOrNil bundle: nibBundleOrNil]) {
        
    }
    return self;
}

- (void)awakeFromNib
{
    NSLog(@"%s", __func__);
    [super awakeFromNib];
}

- (void)viewDidLoad {
    NSLog(@"%s", __func__);
    [super viewDidLoad];
    self.title = @"xib";
}

- (void)viewWillAppear:(BOOL)animated
{
    NSLog(@"%s", __func__);
    [super viewWillAppear: animated];
}

- (void)viewDidAppear:(BOOL)animated
{
    NSLog(@"%s", __func__);
    [super viewDidAppear: animated];
}

- (void)viewWillDisappear:(BOOL)animated
{
    NSLog(@"%s", __func__);
    [super viewWillDisappear: animated];
}

- (void)viewDidDisappear:(BOOL)animated
{
    NSLog(@"%s", __func__);
    [super viewDidDisappear: animated];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end
```

#### 从xib或者从storyboard加载控制器

从 xib 中加载控制器

```
FirstViewController *vc = [[[NSBundle mainBundle] loadNibNamed: @"FirstViewController" owner: nil options: nil] firstObject];
```

一般 xib 中都主要放一个控制器或者view，所以写 firstObject 就可以获取到，如果有多个，根据实际情况获取。单独创建的 xib 文件的时候，设置好拖出来的 View Controller 的 Custom Class 中的 Class。

```
2018-05-29 10:50:37.836332+0800 控制器加载原理[21451:626155] -[FirstViewController init]
2018-05-29 10:50:37.836691+0800 控制器加载原理[21451:626155] -[FirstViewController initWithNibName:bundle:]
2018-05-29 10:50:37.838226+0800 控制器加载原理[21451:626155] -[FirstViewController loadView]
2018-05-29 10:50:37.838529+0800 控制器加载原理[21451:626155] -[FirstViewController viewDidLoad]
2018-05-29 10:50:37.839007+0800 控制器加载原理[21451:626155] -[FirstViewController viewWillAppear:]
2018-05-29 10:50:37.851140+0800 控制器加载原理[21451:626155] -[FirstViewController viewWillLayoutSubviews]
2018-05-29 10:50:37.851257+0800 控制器加载原理[21451:626155] -[FirstViewController viewDidLayoutSubviews]
2018-05-29 10:50:39.292940+0800 控制器加载原理[21451:626155] -[FirstViewController viewWillDisappear:]
2018-05-29 10:50:39.797402+0800 控制器加载原理[21451:626155] -[FirstViewController viewDidDisappear:]
```

从上面的打印可以看出方法的执行过程

* - (instancetype)init;
* - (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil;
* - (void)loadView;
* - (void)viewDidLoad;
* - (void)viewWillAppear:(BOOL)animated;
* - (void)viewWillLayoutSubviews;
* - (void)viewDidLayoutSubviews;
* - (void)viewWillAppear:(BOOL)animated;
* - (void)viewDidAppear:(BOOL)animated;




### View的加载过程

 


