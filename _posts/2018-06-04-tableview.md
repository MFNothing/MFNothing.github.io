---
layout:     post
title:      "tableView 优化的相关问题"
subtitle:   "tableView 优化的相关问题"
date:       2018-06-03 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 优化
    - 响应链
    - UIView
---

## tableView 优化的相关问题

### 数据源同步

如何解决tableView在多线程的情况下修改或者访问数据源的一个同步问题?

两种方式

* 并发访问 & 数据拷贝
* 串行访问

#### 并发访问、数据拷贝

![](/img/in-mpost/TableView/DeleteErrorMake.png)

当我们从主线程拷贝一份数据，并对数据进行新增数据请求时。（这个请求会请求新增数据）

如果用户对数据项进行了一次删除操作。（这个操作会立即执行，但为了不影响用户体验，删除的请求会在另外的子线程执行）

当我们在子线程完成网络请求、数据解析以及预排版等操作后，想要将数据返回给主线程进行刷新。

此时主线程里面的数据与返回回来的数据就出现了问题。

用户刚刚删除的数据又重新出现了。

我们就需要对这样的情况进行处理，记录下用户删除的操作，并在数据返回同步到主线程之前，去判断用户是否进行了删除操作，如果进行了，则还要对返回的数据进行一次删除操作，保证数据的统一。

![](/img/in-mpost/TableView/DeleteCorrect.png)

这样数据就不会因为用户的删除操作而产生数据错乱。

#### 串行访问

这种方式就没有对数据进行拷贝操作，而是直接对原数据进行操作。

![](/img/in-mpost/TableView/Serial.png)

当网络请求返回的时候，数据经过解析后插入给一个串行队列。

期间用户进行了一次删除操作，并将删除操作插入到串行队列中，并等待

串行队列对新增数据进行预排版之后（这个过程会修改原来的数据，向其增加新增的数据）

执行删除操作（这里也对原数据进行了删除操作，因为是串行的，所以数据的访问相对安全）

刷新界面

### 事件传递 & 视图响应

#### CALayer 与 UIView 的关系

* UIView 为 CALayer 提供内容，以及负责处理触摸等事件，参与响应链
* CALayer 负责显示内容 contents

这里用到了六大设计原则之中的单一职责原则，这就是为什么要区分UIView 和 CALayer 之间的工作分工。

六大设计原则：

* 单一职责原则：一个类只负责一个功能领域中的相应职责，或者可以定义为：就一个类而言，应该只有一个引起它变化的原因。
* 开闭原则：一个软件实体应该对拓展开发，对修改关闭。即软件实体应该尽量在不修改原有代码的情况下进行拓展。
* 里式替换原则：所有引用基类的地方必须能够透明地使用其子类的对象。即用子类替换父类可以正常使用。
* 依赖倒置原则：抽象不应该依赖细节，细节应当依赖于抽象。换言之，要针对接口编程，而不是针对实现编程。
* 接口隔离原则：使用多个专门的接口，而不使用单一的总接口，即客户端不应该依赖那些它不需要的接口。
* 迪米特原则：一个软件实体应该尽可能少地与其他实体发生相互作用。

#### 事件传递机制

当我们触控手机屏幕时系统便会将这一操作封装成一个UIEvent放到事件队列里面，然后Application从事件队列取出这个事件，接着需要找到去响应这个事件的最佳视图也就是Responder, 所以开始的第一步应该是找到Responder, 那么又是如何找到的呢？那就不得不引出UIView的2个方法：
 
 * 返回视图层级中能响应触控点的最深视图

	`-(UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event` 
	
 * 返回视图是否包含指定的某个点
  
 	`-(BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event`
 	

当我们点击屏幕的时候，这个事件就会传递给 `UIApplication`，然后由 `UIApplication` 传递给当前的 `UIWindow `，然后 `UIWindow` 会调用 `hitTest:withEvent:` 方法去返回当前被点击的视图。下面就是这个方法的具体实现。

![](/img/in-mpost/TableView/HitTest.png)

* 首先会判断当前视图是否隐藏、是否不可以点击以及透明度是否小于0.01，如果有一项满足，则返回nil，然后由当前视图的父视图遍历其兄弟视图，去调用对应的 `hitTest:withEvent:` 方法。
* 如果视图不隐藏、可以支持点击、并且透明度大于0.01，则调用 `pointInside:withEvent:` 方法判断点击区域是否在当前视图外部，如果返回 `NO`, 表示在外部则返回nil
* 如果在当前视图内部，则倒叙遍历子视图，递归调用 `hitTest:withEvent:` 方法，如果返回nil，则返回当前视图
* 如果返回不为空，则返回子视图中递归返回的视图。

#### 视图响应链

当我们将事件传递给视图之后，就需要让视图去响应，如果当前视图不能响应事件，就会交由下一个视图`nextResponder` 去响应。

通过官方的一个视图来了解视图响应链

![](/img/in-mpost/TableView/ResponderChain.png)

* `UILabel` 和 `UITextField` 、`UIButton` 的下一个响应者是一个 `UIView`
* `UIView` 的下一个响应者可能还是一个 `UIView`
* `UIView` 最终的响应者可能是一个 `UIViewController` 或者 `UIWindow`
* 如果是一个 `UIViewController`，那么它的下一个响应者是 `UIWindow`
* `UIWindow` 的下一个响应者是 `UIApplication` 
* `UIApplication` 的下一个响应者是 `UIApplicationDelegate`

视图响应的方法（UIResponder）

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
```
![](/img/in-mpost/TableView/Touch.png)

当我们点击白色的圆点区域时，事件会传递到 C2，如果 C2 不响应会传递给 B2，如果 B2 也不响应会传递给 A，然后依次传递，直至传递到 `UIApplicationDelegate`，然后系统就会丢弃这个事件，当什么都没有发生。

### UI图像显示原理

深入了解可以看一个 [这个大神的文章](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

下面的内容也会有部分来自上面的文章

#### 简单了解

![](/img/in-mpost/TableView/ios_screen_display.png)

通常来说，计算机系统中 CPU、GPU、显示器是以上面这种方式协同工作的。CPU 计算好显示内容提交到 GPU，GPU 渲染完成后将渲染结果放入帧缓冲区，随后视频控制器会按照 VSync 信号逐行读取帧缓冲区的数据，经过可能的数模转换传递给显示器显示。

在最简单的情况下，帧缓冲区只有一个，这时帧缓冲区的读取和刷新都都会有比较大的效率问题。为了解决效率问题，显示系统通常会引入两个缓冲区，即双缓冲机制。在这种情况下，GPU 会预先渲染好一帧放入一个缓冲区内，让视频控制器读取，当下一帧渲染好后，GPU 会直接把视频控制器的指针指向第二个缓冲器。如此一来效率会有很大的提升。

双缓冲虽然能解决效率问题，但会引入一个新的问题。当视频控制器还未读取完成时，即屏幕内容刚显示一半时，GPU 将新的一帧内容提交到帧缓冲区并把两个缓冲区进行交换后，视频控制器就会把新的一帧数据的下半段显示到屏幕上，造成画面撕裂现象。

为了解决这个问题，GPU 通常有一个机制叫做垂直同步（简写也是 V-Sync），当开启垂直同步后，GPU 会等待显示器的 VSync 信号发出后，才进行新的一帧渲染和缓冲区更新。这样能解决画面撕裂现象，也增加了画面流畅度，但需要消费更多的计算资源，也会带来部分延迟。

那么目前主流的移动设备是什么情况呢？从网上查到的资料可以知道，iOS 设备会始终使用双缓存，并开启垂直同步。而安卓设备直到 4.1 版本，Google 才开始引入这种机制，目前安卓系统是三缓存+垂直同步。

#### UIView 的绘制流程

![](/img/in-mpost/TableView/iOS_UIView_Display.png)

上图就展示了一个绘制流程。

当我们创建一个 UIView 控件之后，它的显示部分是交由 CALayer的，CALayer 中有一个 contents 属性，就是我们要绘制到屏幕上的一个位图。比如说我们创建的是一个显示 `hello world` 的 UILabel。那么 contents 里面放置就是一个显示 `hello world` 的位图，然后系统会在合适的时机回调给我们 `drawRect:` 方法，让我们可以进一步进行绘制。绘制好的位图会交由 Core Animation这个框架交由 OpenGL 渲染管线，显示到屏幕上。

这包括两个部分：CPU和GPU的工作

#### CPU工作

* Layout：UI 的布局和文本的计算
* Display：绘制（比如说 `drawRect:` 方法）
* 做一些准备工作：图片的编解码
* 提交给 GPU：提交位图

#### GPU渲染管线过程

* 顶点着色
* 图元装配
* 光栅化
* 片段着色
* 片段处理

当GPU渲染完成之后，会将新一帧的内容提交到帧缓存区中，待下一个垂直同步信号发出后，进行新的一帧渲染和缓冲区更新。

### UI卡顿&掉帧的原因

![](/img/in-mpost/TableView/ios_frame_drop.png)

一般页面滑动的流畅性是60fps，每秒会有60帧的画面更新，也就是每16.7ms需要产生一帧画面，这过程需要GPU，CPU协同产生一帧数据。

在 VSync 信号到来后，系统图形服务会通过 CADisplayLink 等机制通知 App，App 主线程开始在 CPU 中计算显示内容，比如视图的创建、布局计算、图片解码、文本绘制等。随后 CPU 会将计算好的内容提交到 GPU 去，由 GPU 进行变换、合成、渲染。随后 GPU 会把渲染结果提交到帧缓冲区去，等待下一次 VSync 信号到来时显示到屏幕上。由于垂直同步的机制，如果在一个 VSync 时间内，CPU 或者 GPU 没有完成内容提交，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏会保留之前的内容不变。这就是界面卡顿的原因。（一个 VSync 时间是16.7ms）

从上面的图中可以看到，CPU 和 GPU 不论哪个阻碍了显示流程，都会造成掉帧现象。所以开发时，也需要分别对 CPU 和 GPU 压力进行评估和优化。

#### 滑动优化方案

从上面我们知道我们可以从CPU和GPU上进行优化

CPU 耗时的原因主要包括：

* 对象（创建、调整、销毁）
* 预排版（布局计算、Autolayout、文本计算）
* 预渲染（文本、图片等异步绘制、图片编解码）

优化：

* 对于对象部分，我们可以放到子线程中去进行
* 预排版中的文本计算和布局计算也可以放到子线程中去做。

GPU 耗时的原因主要包括：

* 纹理的渲染：所有的 Bitmap，包括图片、文本、栅格化的内容，最终都要由内存提交到显存，绑定为 GPU Texture。不论是提交到显存的过程，还是 GPU 调整和渲染 Texture 的过程，都要消耗不少 GPU 资源。
* 视图的混合：当多个视图（或者说 CALayer）重叠在一起显示时，GPU 会首先把他们混合到一起。如果视图结构过于复杂，混合的过程也会消耗很多 GPU 资源。
* 图形的生成：CALayer 的 border、圆角、阴影、遮罩（mask），CASharpLayer 的矢量图形显示，通常会触发离屏渲染（offscreen rendering），而离屏渲染通常发生在 GPU 中。

优化：

* 纹理渲染：避免这种情况的方法只能是尽量减少在短时间内大量图片的显示，尽可能将多张图片合成为一张进行显示。
* 视图混合：为了减轻这种情况的 GPU 消耗，应用应当尽量减少视图数量和层次，并在不透明的视图里标明 opaque 属性以避免无用的 Alpha 通道合成。当然，这也可以用上面的方法，把多个视图预先渲染为一张图片来显示。
* 最彻底的解决办法，就是把需要显示的图形在后台线程绘制为图片，避免使用圆角、阴影、遮罩等属性。

### UI绘制原理&异步绘制

UIView 的 CALayer 的 delegate 默认是自身。

#### 绘制流程

![](/img/in-mpost/TableView/displayLayer.png)

当用户调用 `[UIView setNeedsDisplay]` 会立刻调用其 layer 的 `[view.layer setNeedsDisplay]` 方法，然后会在此时打一个标记，到当前 runloop 结束时，会调用 `[view.layer display]` 方法，才开始介入到 UI 视图的绘制流程中。

在 `[view.layer display]` 方法中，首先会去判断 view.layer 的代理是否响应 `displayLayer:` 方法，如果不响应则走系统绘制流程。如果响应，我们可以从这个方法中去进行异步绘制。

#### 系统绘制流程

![](/img/in-mpost/TableView/displayLayer.png)

在 CALayer 内部会创建一个 backing store（可以理解为一个CGContext），然后会去判断 layer 是否有代理，如果没有代理，会去调用 `[CALayer drawInContext:]` 方法。

如果有代理会去调用 `[layer.delegate drawLayer:inContext]` 方法，然后做当前视图的绘制工作，这个过程在内部过程中的，然后调用 `UIView drawRect:` 方法，这个方法默认是什么都不做，通过这个方法，我们可以在系统绘制的基础之上再进行其他的一些绘制操作。

最后两个分支的都会由 CALayer 上传 backing store 到 GPU，结束绘制流程。

#### 异步绘制

当我们实现了 view.layer 的 `displayLayer:` 方法，那么就会交由代理去生成对应的位图（bitmap），同时需要设置该位图作为 layer.contents 属性的值。

```
/*
 * data                 指向要渲染的绘制内存的地址。这个内存块的大小至少是（bytesPerRow*height）个字节。使用时可填NULL或unsigned char类型的指针。
 * width                bitmap的宽度,单位为像素
 * height               bitmap的高度,单位为像素
 * bitsPerComponent     内存中像素的每个组件的位数.例如，对于32位像素格式和RGB 颜色空间，你应该将这个值设为8。
 * bytesPerRow          bitmap的每一行在内存所占的比特数，一个像素一个byte。
 * colorspace           bitmap上下文使用的颜色空间。
 * bitmapInfo           指定bitmap是否包含alpha通道，像素中alpha通道的相对位置，像素组件是整形还是浮点型等信息的字符串。
 */

- (void)displayLayer:(CALayer *)layer
{
    NSLog(@"%s", __func__);
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        const CGSize size = CGSizeMake(200, 200);
        const size_t bitsPerComponent = 8;
        const size_t bytesPerRow = size.width * 4;
       CGContextRef context =  CGBitmapContextCreate(calloc(sizeof(unsigned char), bytesPerRow * size.height), size.width, size.height, bitsPerComponent, bytesPerRow, CGColorSpaceCreateDeviceRGB(), kCGImageAlphaPremultipliedLast);
        CGContextAddArc(context, 50, 50, 30, 0, M_PI * 2, 0);
        CGContextSetRGBFillColor(context, 0, 1, 0, 1);
        CGContextFillPath(context);
        CGImageRef imageRef = CGBitmapContextCreateImage(context);
        layer.contents = (__bridge id)imageRef;
        layer.backgroundColor = [UIColor yellowColor].CGColor;
    });
}
```

### UI离屏渲染

离屏渲染指的是 GPU 在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作

#### 何时触发离屏渲染

* 圆角和 maskToBounds 一起使用时
* 图层蒙版
* 阴影
* 光栅化

#### 为什么要避免离屏渲染

在触发离屏渲染的时候，会增加GPU的工作量，导致掉帧卡顿的情况。

