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
    - UIViewController
---

## 控制器view加载原理

### 几种与视图相关创建控制器的方式

**创建跟控制器相同名称的xib文件**

![](/img/in-mpost/Controller-Loading-Principle/CreateController1.gif)

```
FirstViewController *vc = [[FirstViewController alloc] initWithNibName: @"FirstViewController" bundle: nil];
```

**先创建一个控制器，然后将一个xib的view与控制器的view关联**

![](/img/in-mpost/Controller-Loading-Principle/CreateController2.gif)

这个方式跟第一个创建方式差不多，只是第一种方式默认做了第二种关联 view 的操作。注意这里没有直接使用 init 方法创建。虽然 init 方法还是会调用 initWithNibName:bundle: 方法，但是它传入的 nibname 是为空的，当我们重写 loadView 的时候，它不会去找与 xxxxController控制器名字相同的 xxxxController.xib 文件了，也不会去找 xxxx.xib 文件了。

```
SecondViewController *vc = [[SecondViewController alloc] initWithNibName: @"SecondViewController" bundle: nil];
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


### 自定义一个控制器 和 view

我们通过自定义一个控制器并重写它的一些方法，打印一些视图加载的信息。

```
#import <UIKit/UIKit.h>

@interface FirstViewController : UIViewController

@end

#import "FirstViewController.h"

@interface FirstViewController ()

@end

@implementation FirstViewController

- (void)loadView
{
    NSLog(@"%s", __func__);
    [super loadView];
}

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

```
#import <UIKit/UIKit.h>

@interface FirstView : UIView

@end

#import "FirstView.h"

@implementation FirstView

- (void)awakeFromNib
{
    NSLog(@"%s", __func__);
    [super awakeFromNib];
}

- (instancetype)initWithCoder:(NSCoder *)aDecoder
{
    NSLog(@"%s", __func__);
    if (self = [super initWithCoder: aDecoder]) {
        
    }
    return self;
}

@end
```

#### 理解 awakeFromNib

从xib或者storyboard加载完毕就会调用。

当我们通过第一种和第二种方式创建控制器时，我们其实只是通过 xib 加载了一个控制器的 view。

当我们通过第三种和第四种方式创建控制器时，我们让 xib 和 storyboard 根据内容样式加载了控制器的 view 和 控制器。

所以通过第一种或第二种方式创建的 view，不会去响应控制器的 awakeFromNib 方法。但是如果你将 xib 中的那个 view 的 Custom Class 的 Class 指定，并重写 awakeFromNib 方法，你可以在指定的那个类中，看到它被调用。

#### 理解 initWithCoder:

只要对象是从文件解析来的，就会调用。

如果 awakeFromNib 和 initWithCoder: 方法同时存在，会先调用 initWithCoder: 方法。

#### 理解 loadView 

你不应该直接调用这个方法。

先要知道控制器的 view 是通过懒加载的方式进行加载的。当我们调用 self.view 的时候，会在 get 方法中判断 view 是否为空，如果为空就会调用控制器的 loadView 方法。这个方法并不是只会调用一次的，当内存紧张的时候，系统会将已经加载的但是此刻没有展示的 view 先释放掉，当你 pop 或者回到这个页面的时候，才重新加载出来。

所以 loadView 方法会初始化控制器的 view。

这个 view 会根据我们有没有在 initWithNibName:bundle: 方法指定 nibName，如果有回去找指定的xib，如果没有回去找跟控制器类名称相同的 xib 文件（如果控制器类叫 FirstViewController，那么它会去找 FirstViewController.xib 文件）。如果还没有找到就去找名字去掉 Controller 之后跟类名一样的 xib 文件（如 FirstView.xib）。如果还没有找到会创建一个简单的 UIView 作为控制器的 view。

当然这个前提是你没有重写 loadView 方法。如果重写还是会走自定义方法的。所以当你重写这个方法的时候就不用 [super loadView] 了。如果你只是为了打印它是否调用可以加上。

#### 理解 viewDidLoad 

在控制器的视图被加载到内存后调用。只是加载到内存，并没有加到window上面，你在此时打印 [self.view superview] 返回是为空的。要到 viewWillLayoutSubviews 时及之后才会不为空。

当我们通过第一种和第二种方式创建控制器时，我们使用与 xib 尺寸不同的设备（模拟器或真机），打印 self.view 的frame 跟我们机型是不符的，原因是我的 view 在此刻还没有加载出来，所以以这种方式去加载控制器，我们最好在 viewDidLayoutSubviews 或者 viewWillLayoutSubviews 方法来写子控件的 frame，如果你不通过约束的话。

#### 几种创建方式的打印

**第一种方式**

```
2018-05-30 16:29:18.982103+0800 控制器加载原理[57210:1973950] -[FirstViewController initWithNibName:bundle:]
2018-05-30 16:29:18.982411+0800 控制器加载原理[57210:1973950] ------Before Push Or Present------
2018-05-30 16:29:18.983351+0800 控制器加载原理[57210:1973950] -[FirstViewController loadView]
2018-05-30 16:29:18.985113+0800 控制器加载原理[57210:1973950] -[FirstView initWithCoder:]
2018-05-30 16:29:18.986110+0800 控制器加载原理[57210:1973950] -[FirstView awakeFromNib]
2018-05-30 16:29:18.986314+0800 控制器加载原理[57210:1973950] -[FirstViewController viewDidLoad]
2018-05-30 16:29:18.988668+0800 控制器加载原理[57210:1973950] -[FirstViewController viewWillAppear:]
2018-05-30 16:29:18.990660+0800 控制器加载原理[57210:1973950] -[FirstViewController viewWillLayoutSubviews]
2018-05-30 16:29:18.990834+0800 控制器加载原理[57210:1973950] -[FirstViewController viewDidLayoutSubviews]
2018-05-30 16:29:19.493277+0800 控制器加载原理[57210:1973950] -[FirstViewController viewDidAppear:]
2018-05-30 16:29:25.498040+0800 控制器加载原理[57210:1973950] -[FirstViewController viewWillDisappear:]
2018-05-30 16:29:26.001019+0800 控制器加载原理[57210:1973950] -[FirstViewController viewDidDisappear:]
```

**第二种方式**

```
2018-05-30 16:28:24.431892+0800 控制器加载原理[57176:1972623] -[SecondViewController initWithNibName:bundle:]
2018-05-30 16:28:24.432174+0800 控制器加载原理[57176:1972623] ------Before Push Or Present------
2018-05-30 16:28:24.432911+0800 控制器加载原理[57176:1972623] -[SecondViewController loadView]
2018-05-30 16:28:24.433695+0800 控制器加载原理[57176:1972623] -[SecondView initWithCoder:]
2018-05-30 16:28:24.434499+0800 控制器加载原理[57176:1972623] -[SecondView awakeFromNib]
2018-05-30 16:28:24.434699+0800 控制器加载原理[57176:1972623] -[SecondViewController viewDidLoad]
2018-05-30 16:28:24.437939+0800 控制器加载原理[57176:1972623] -[SecondViewController viewWillAppear:]
2018-05-30 16:28:24.439937+0800 控制器加载原理[57176:1972623] -[SecondViewController viewWillLayoutSubviews]
2018-05-30 16:28:24.440112+0800 控制器加载原理[57176:1972623] -[SecondViewController viewDidLayoutSubviews]
2018-05-30 16:28:24.942532+0800 控制器加载原理[57176:1972623] -[SecondViewController viewDidAppear:]
2018-05-30 16:28:47.395982+0800 控制器加载原理[57176:1972623] -[SecondViewController viewWillDisappear:]
2018-05-30 16:28:47.900161+0800 控制器加载原理[57176:1972623] -[SecondViewController viewDidDisappear:]
``` 

**第三种方式**

```
2018-05-30 16:29:49.105215+0800 控制器加载原理[57240:1974927] -[ThirdViewController initWithCoder:]
2018-05-30 16:29:49.105420+0800 控制器加载原理[57240:1974927] -[ThirdView initWithCoder:]
2018-05-30 16:29:49.106315+0800 控制器加载原理[57240:1974927] -[ThirdViewController awakeFromNib]
2018-05-30 16:29:49.106432+0800 控制器加载原理[57240:1974927] -[ThirdViewController viewDidLoad]
2018-05-30 16:29:49.106622+0800 控制器加载原理[57240:1974927] -[ThirdView awakeFromNib]
2018-05-30 16:29:49.106803+0800 控制器加载原理[57240:1974927] ------Before Push Or Present------
2018-05-30 16:29:49.109713+0800 控制器加载原理[57240:1974927] -[ThirdViewController viewWillAppear:]
2018-05-30 16:29:49.111661+0800 控制器加载原理[57240:1974927] -[ThirdViewController viewWillLayoutSubviews]
2018-05-30 16:29:49.111820+0800 控制器加载原理[57240:1974927] -[ThirdViewController viewDidLayoutSubviews]
2018-05-30 16:29:49.613662+0800 控制器加载原理[57240:1974927] -[ThirdViewController viewDidAppear:]
2018-05-30 16:29:51.016962+0800 控制器加载原理[57240:1974927] -[ThirdViewController viewWillDisappear:]
2018-05-30 16:29:51.520294+0800 控制器加载原理[57240:1974927] -[ThirdViewController viewDidDisappear:]
```

**第四种方式**

```
2018-05-30 16:30:58.891572+0800 控制器加载原理[57280:1976590] -[FourthViewController initWithCoder:]
2018-05-30 16:30:58.891882+0800 控制器加载原理[57280:1976590] -[FourthViewController awakeFromNib]
2018-05-30 16:30:58.892043+0800 控制器加载原理[57280:1976590] ------Before Push Or Present------
2018-05-30 16:30:58.892652+0800 控制器加载原理[57280:1976590] -[FourthViewController loadView]
2018-05-30 16:30:58.893230+0800 控制器加载原理[57280:1976590] -[FourthView initWithCoder:]
2018-05-30 16:30:58.894314+0800 控制器加载原理[57280:1976590] -[FourthView awakeFromNib]
2018-05-30 16:30:58.894603+0800 控制器加载原理[57280:1976590] -[FourthViewController viewDidLoad]
2018-05-30 16:30:58.896784+0800 控制器加载原理[57280:1976590] -[FourthViewController viewWillAppear:]
2018-05-30 16:30:58.898938+0800 控制器加载原理[57280:1976590] -[FourthViewController viewWillLayoutSubviews]
2018-05-30 16:30:58.899125+0800 控制器加载原理[57280:1976590] -[FourthViewController viewDidLayoutSubviews]
2018-05-30 16:30:59.400344+0800 控制器加载原理[57280:1976590] -[FourthViewController viewDidAppear:]
2018-05-30 16:30:59.718999+0800 控制器加载原理[57280:1976590] -[FourthViewController viewWillDisappear:]
2018-05-30 16:31:00.221843+0800 控制器加载原理[57280:1976590] -[FourthViewController viewDidDisappear:]
```

**一个纯代码创建的ViewController**

```
2018-05-30 16:34:45.833374+0800 控制器加载原理[57418:1981699] -[FifthViewController init]
2018-05-30 16:34:45.833563+0800 控制器加载原理[57418:1981699] -[FifthViewController initWithNibName:bundle:]
2018-05-30 16:34:45.833785+0800 控制器加载原理[57418:1981699] ------Before Push Or Present------
2018-05-30 16:34:45.834503+0800 控制器加载原理[57418:1981699] -[FifthViewController loadView]
2018-05-30 16:34:45.834857+0800 控制器加载原理[57418:1981699] -[FifthViewController viewDidLoad]
2018-05-30 16:34:45.837540+0800 控制器加载原理[57418:1981699] -[FifthViewController viewWillAppear:]
2018-05-30 16:34:45.839418+0800 控制器加载原理[57418:1981699] -[FifthViewController viewWillLayoutSubviews]
2018-05-30 16:34:45.839549+0800 控制器加载原理[57418:1981699] -[FifthViewController viewDidLayoutSubviews]
2018-05-30 16:34:46.340555+0800 控制器加载原理[57418:1981699] -[FifthViewController viewDidAppear:]
2018-05-30 16:34:54.223745+0800 控制器加载原理[57418:1981699] -[FifthViewController viewWillDisappear:]
2018-05-30 16:34:54.727455+0800 控制器加载原理[57418:1981699] -[FifthViewController viewDidDisappear:]
```

除开第三种方式，我们在 viewDidLoad 后面的执行过程基本一致。第三种与第一、二、四种的区别在于，控制器在加载时它的 view 是否已经创建成功了，第三种在我们加载之前就已经加载成功了。其他都是经过 loadView 方法去初始化 view 的。

注意一个小细节，就当我们 push 的时候，比如 A push 到 B 的时候， A 的 viewDidDisappear: 方法会在 B 的 viewDidLayoutSubviews 方法执行之后执行。

present 方法，A 的 viewDidDisappear: 方法会在 B 的 viewDidAppear: 方法之后执行。

### View的加载过程

#### 定义一个 View 


```
#import <UIKit/UIKit.h>

@interface ThirdView : UIView

@end

#import "ThirdView.h"

@implementation ThirdView

- (void)awakeFromNib
{
    NSLog(@"%s", __func__);
    [super awakeFromNib];
}

- (instancetype)initWithCoder:(NSCoder *)aDecoder
{
    NSLog(@"%s", __func__);
    if (self = [super initWithCoder: aDecoder]) {
        
    }
    return self;
}

- (instancetype)init
{
    NSLog(@"%s", __func__);
    if (self = [super init]) {
        
    }
    return self;
}

- (instancetype)initWithFrame:(CGRect)frame
{
    NSLog(@"%s", __func__);
    if (self = [super initWithFrame:frame]) {
        
    }
    return self;
}

- (void)layoutSubviews
{
    NSLog(@"%s", __func__);
    [super layoutSubviews];
}

- (void)drawRect:(CGRect)rect
{
    NSLog(@"%s", __func__);
    [super drawRect:rect];
}

@end
```

#### 几种创建View的方式

**纯代码**

```
ThirdView *thirdView = [[ThirdView alloc] init];
```

```
2018-05-30 17:00:36.887514+0800 控制器加载原理[58175:2012242] -[ThirdView init]
2018-05-30 17:00:36.887651+0800 控制器加载原理[58175:2012242] -[ThirdView initWithFrame:]
2018-05-30 17:00:36.887989+0800 控制器加载原理[58175:2012242] ------Before add------
2018-05-30 17:00:36.893572+0800 控制器加载原理[58175:2012242] -[ThirdView layoutSubviews]
2018-05-30 17:00:36.894210+0800 控制器加载原理[58175:2012242] -[ThirdView drawRect:]
```

**xib**

```
ThirdView *thirdView = [[[NSBundle mainBundle] loadNibNamed: @"ThirdView" owner: nil options: nil] firstObject];
```

```
2018-05-30 16:59:55.715950+0800 控制器加载原理[58147:2011103] -[ThirdView initWithCoder:]
2018-05-30 16:59:55.716275+0800 控制器加载原理[58147:2011103] -[ThirdView awakeFromNib]
2018-05-30 16:59:55.716491+0800 控制器加载原理[58147:2011103] ------Before add------
2018-05-30 16:59:55.722078+0800 控制器加载原理[58147:2011103] -[ThirdView layoutSubviews]
2018-05-30 16:59:55.722611+0800 控制器加载原理[58147:2011103] -[ThirdView drawRect:]
```

#### layoutSubviews 和 drawRect: 什么时候调用

layoutSubviews 跟上面一样，在这里设置子控件的位置最好，如果你没有设置约束的话。init 和 initWithFrame 的时候还没有添加到视图中。

drawRect: 的时候可以绘制一些图像，这个时候回默认有一个上下文给你绘制图像，如果不在这个时候需要 push 一个上下文进行绘制。


我们调用 setNeedsDisplay 方法的时候，会在下一个 runloop 的时候执行  drawRect: 方法。

setNeedsLayout 方法也一样，会在下一个 runloop 的时候执行 layoutSubviews 方法。如果想要在当前 runloop 中执行，需要再执行 layoutIfNeeded 方法。

这几个都是 UIView 的方法。





