---
layout:     post
title:      "百度地图SDK接入与使用"
subtitle:   "Baidu Maps SDK"
date:       2018-03-10 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Map
---

## 百度地图SDK接入与使用

### 一、接入部分

[官方文档地址](http://lbsyun.baidu.com/index.php?title=iossdk) 

#### cocoapods集成

在 Podfile 中写入下面的代码

```
platform :ios, '8.0'
target '你的项目名字' do
  pod 'BaiduMapKit'
end

```

**pod repo update** 更新整个pod的在本地的库的版本，会跟网有关，很慢

**pod update** 更新的时候会去查本地有没有更新，所以可能pod update的时候，本地的库版本并没有更新，导致库没有被更新到最新版本

#### 手动安装

1. 下载SDK源文件 [BaiduMap](http://lbsyun.baidu.com/index.php?title=iossdk/sdkiosdev-download) 
2. 百度地图 iOS SDK 采用分包的形式提供 .framework包，请广大开发者使用时确保各分包的版本保持一致。将所需的BaiduMapAPI_**.framework拷贝或者拖拽到工程所在文件夹下，功能包内容如下：

	|开发包文件|功能包内容|
	|:-------:|:---------:|
	|BaiduMapAPI_Base.framework|为基础包，使用SDK任何功能都需导入，其他分包可按需导入。|
	|BaiduMapAPI_Map.framework	|地图功能包|
	|BaiduMapAPI_Search.framework	|检索功能包|
	|BaiduMapAPI_Cloud.framework	|云检索功能包|
	|BaiduMapAPI_Utils.framework	|工具功能包|
	|BaiduMapAPI_Radar.framework	|周边雷达工具包|
	|BaiduMapAPI_Location.framework|	定位功能|
	|thirdlibs	|第三方openssl静态库用于支持https，版本v1.0.2e|

3. 静态库中采用Objective-C++实现，因此需要您保证您工程中至少有一个.mm后缀的源文件(您可以将任意一个.m后缀的文件改名为.mm)
4. 需要引入的系统库文件
百度地图SDK中提供了定位功能和动画效果，v2.0.0版本开始使用OpenGL渲染，因此您需要在您的Xcode工程需引入的系统库如下表所示：

	|库名称	|备注|
	|:-------:|:---------:|
	|CoreLocation.framework|	|
	|QuartzCore.framework||	
	|OpenGLES.framework	||	
	|SystemConfiguration.framework	|统计信息，用于改善产品|
	|CoreGraphics.framework	 ||	
	|Security.framework	|是为了统计app信息使用|
	|libsqlite3.0.tbd（xcode7以前为 libsqlite3.0.dylib）|		v2.9.0新增的系统库，使用v2.9.0及以上版本的地图SDK，务必增加 |
	|CoreTelephony.framework	|v2.9.0新增的系统库，使用v2.9.0及以上版本的地图SDK，务必增加|
	|libstdc++.6.0.9.tbd（xcode7以前为libstdc++.6.0.9.dylib）	|v2.9.0新增的系统库，使用v2.9.0及以上版本的地图SDK，务必增加|

	添加方法： 在Xcode的Project -> Active Target ->Build Phases ->Link Binary With Libraries，添加这几个系统库即可。


5. 环境配置
在TARGETS->Build Settings->Other Linker Flags 中添加-ObjC，字母O和C大写。
6. 从BaiduMapAPI_Map.framework||Resources文件中选择mapapi.bundle文件，并勾选“Copy items if needed”复选框，单击“Add”按钮，将资源文件添加到工程中。

#### 创建API密钥并添加到您的项目中

查找或创建密钥[地址](http://lbsyun.baidu.com/apiconsole/key)

密钥使用，在AppDelegate.m中添加

```
#import <BaiduMapAPI_Map/BMKMapComponent.h>

- (void)setBaiduMap
{
    BMKMapManager *baiduMapManager = [[BMKMapManager alloc] init];
    BOOL ret = [baiduMapManager start: @"lcB114xaksAShwuAEQOvEGWjI99GITD7" generalDelegate: nil];
    if (ret == NO) {
        NSLog(@"manager start failed!");
    }
}
```

### 二、地图使用

#### 地图显示

```
#import <BaiduMapAPI_Map/BMKMapView.h>

@property (nonatomic, strong) BMKMapView *mapView;
- (void)initBaiduMap
{
    self.mapView = [[BMKMapView alloc]initWithFrame:self.view.bounds];
    self.mapView.zoomLevel = 18;
    self.mapView.centerCoordinate = CLLocationCoordinate2DMake(30.538748, 104.062634);
    self.mapView.delegate = self; // 此处记得不用的时候需要置nil，否则影响内存的释放
    [self.view addSubview: self.mapView];
}

```

#### 定位

首先需要在 info.plist 添加 NSLocationWhenInUseUsageDescription 或者 NSLocationAlwaysUsageDescription 。根据自己的需求添加。

然后添加以下代码请求用户授权开启定位服务

```
#import <BaiduMapAPI_Map/BMKMapView.h>
#import <BaiduMapAPI_Location/BMKLocationComponent.h>
@interface ViewController () <BMKMapViewDelegate, BMKLocationServiceDelegate>
@property (nonatomic, strong) BMKMapView *mapView;
@property (nonatomic, strong) BMKLocationService *baiduLocationService;
@property (nonatomic, assign) CLLocationCoordinate2D myBaiduLocation;
@end

- (void)initBaiduLoactionManager
{
    self.baiduLocationService = [[BMKLocationService alloc] init];
    self.baiduLocationService.delegate = self;
    [self.baiduLocationService startUserLocationService];
    self.mapView.userTrackingMode = BMKUserTrackingModeNone;
    [self.baiduLocationService startUserLocationService];
}

#pragma mark - BMKLocationServiceDelegate
- (void)didUpdateUserHeading:(BMKUserLocation *)userLocation
{
    //    NSLog(@"heading is %@",userLocation.heading);
    self.myBaiduLocation = userLocation.location.coordinate;
    self.mapView.centerCoordinate = self.myBaiduLocation;
    [self.mapView updateLocationData: userLocation];
    [self.baiduLocationService stopUserLocationService];
}
//处理位置坐标更新
- (void)didUpdateBMKUserLocation:(BMKUserLocation *)userLocation
{
    //    NSLog(@"didUpdateUserLocation lat %f,long %f",userLocation.location.coordinate.latitude,userLocation.location.coordinate.longitude);
    self.myBaiduLocation = userLocation.location.coordinate;
    [self.mapView updateLocationData: userLocation];
    self.mapView.centerCoordinate = self.myBaiduLocation;
    [self.baiduLocationService stopUserLocationService];
}
```

### 显示标记

 
 
**创建一个简单的marker**

继承 BMKPointAnnotation 类，方便利用BMKMapViewDelegate协议中的方法设置标记样式，不实现协议就是默认大头针样式

```
#import <BaiduMapAPI_Map/BMKMapComponent.h>
@interface MINNormalAnnotation : BMKPointAnnotation
@end
@implementation MINNormalAnnotation
@end
```
跟谷歌地图一样，可以设置title和subtitle（谷歌地图的是snippet），默认气泡的标题和子标题

```
/// 要显示的标题；注意：如果不设置title,无法点击annotation,也无法使用回调函数；
@property (copy) NSString *title;
/// 要显示的副标题
@property (copy) NSString *subtitle;
```

```
- (void)addBaiduMarker
{
    MINNormalAnnotation *normalAnnotation = [[MINNormalAnnotation alloc] init];
    CLLocationCoordinate2D coor;
    coor.latitude = 30.541348;
    coor.longitude = 104.062550;
    normalAnnotation.coordinate = coor;
    normalAnnotation.title = @"title";
    normalAnnotation.subtitle = @"subTitle";
    [self.mapView addAnnotation: normalAnnotation];
}
```

添加如下所示代码添加标注数据对象。

**删除标记** 

```
[self.mapView removeAnnotation: normalAnnotation]; // 上面创建的标记
[self.mapView removeAnnotations: self.mapView.annotations]; // 清除所有的标记
- (void)addAnnotations:(NSArray *)annotations; // 批量添加
```

**定制标记图像**

如果您希望更改默认标记图像，您可以通过实现BMKMapViewDelegate的代理方法，这里通过annotation的类名去判断是不是需要定制的标记。

```
#pragma mark - BMKMapViewDelegate
- (BMKAnnotationView *)mapView:(BMKMapView *)mapView viewForAnnotation:(id<BMKAnnotation>)annotation
{
    if ([annotation isKindOfClass: [MINNormalAnnotation class]]) {
        NSString *AnnotationViewID = @"NormalAnnationView";
        BMKPinAnnotationView *annotationView = [[BMKPinAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:AnnotationViewID];
        annotationView.image = [UIImage imageNamed: @"车图-4"];
        return annotationView;
    }
    return nil;
}
```

**自定义气泡**

百度地图的气泡的View是可以用户进行点击的，所以直接设定你想要的View就可以了

```
#pragma mark - BMKMapViewDelegate
- (BMKAnnotationView *)mapView:(BMKMapView *)mapView viewForAnnotation:(id<BMKAnnotation>)annotation
{
    if ([annotation isKindOfClass: [MINNormalAnnotation class]]) {
        NSString *AnnotationViewID = @"NormalAnnationView";
        BMKPinAnnotationView *annotationView = [[BMKPinAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:AnnotationViewID];
        annotationView.image = [UIImage imageNamed: @"车图-4"];
        annotationView.paopaoView = [[BMKActionPaopaoView alloc] initWithCustomView: [[MINPaopaoView alloc] initWithFrame: CGRectMake(0, 0, 100, 50)] ];
        return annotationView;
    }
    return nil;
}
```
相关设置属性

```
///默认情况下, annotation view的中心位于annotation的坐标位置，可以设置centerOffset改变view的位置，正的偏移使view朝右下方移动，负的朝左上方，单位是像素
@property (nonatomic) CGPoint centerOffset;

///默认情况下, 弹出的气泡位于view正中上方，可以设置calloutOffset改变view的位置，正的偏移使view朝右下方移动，负的朝左上方，单位是像素
@property (nonatomic) CGPoint calloutOffset;
```
### 覆盖物（围栏）

**添加或删除覆盖物**

```
/**
 *向地图窗口添加Overlay，需要实现BMKMapViewDelegate的-mapView:viewForOverlay:函数来生成标注对应的View
 *@param overlay 要添加的overlay
 */
- (void)addOverlay:(id <BMKOverlay>)overlay;

/**
 *向地图窗口添加一组Overlay，需要实现BMKMapViewDelegate的-mapView:viewForOverlay:函数来生成标注对应的View
 *@param overlays 要添加的overlay数组
 */
- (void)addOverlays:(NSArray *)overlays;

/**
 *移除Overlay
 *@param overlay 要移除的overlay
 */
- (void)removeOverlay:(id <BMKOverlay>)overlay;

/**
 *移除一组Overlay
 *@param overlays 要移除的overlay数组
 */
- (void)removeOverlays:(NSArray *)overlays;
```

#### 圆

```
@property (nonatomic, assign) CLLocationCoordinate2D circleCoordinate;

- (void)addBaiduCircleMarkerWithRadius:(NSString *)radius
{
    CGFloat radiusNum = [radius floatValue];
    BMKCircle *circle = [BMKCircle circleWithCenterCoordinate: self.circleCoordinate radius: radiusNum];
    [self.mapView addOverlay: circle];
}

#pragma mark - BMKMapViewDelegate
//根据overlay生成对应的View
- (BMKOverlayView *)mapView:(BMKMapView *)mapView viewForOverlay:(id <BMKOverlay>)overlay{
    if ([overlay isKindOfClass:[BMKCircle class]]){
        BMKCircleView* circleView = [[BMKCircleView alloc] initWithOverlay:overlay];
        circleView.fillColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
        circleView.strokeColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
        return circleView;
    }
    return nil;
}
```

#### 线段

```
// CLLocationCoordinate2D是一个结构体，不能保存到数组头
@interface MINCoordinateObject : NSObject
@property (nonatomic, assign) CLLocationCoordinate2D coordinate;
@end
@implementation MINCoordinateObject
@end
@property (nonatomic, strong) NSMutableArray *pathleCoordinateArr;

- (void)addBaiduPath
{
    //记得先清空地图
    CLLocationCoordinate2D coords[self.pathleCoordinateArr.count];
    for (int i = 0; i < self.pathleCoordinateArr.count; i++) {
        MINCoordinateObject *obj = self.pathleCoordinateArr[i];
        coords[i] = obj.coordinate;
    }
    BMKPolyline *polygon = [BMKPolyline polylineWithCoordinates:coords count: self.pathleCoordinateArr.count];
    [self.mapView addOverlay:polygon];
}

#pragma mark - BMKMapViewDelegate
- (BMKOverlayView *)mapView:(BMKMapView *)mapView viewForOverlay:(id <BMKOverlay>)overlay{
    if ([overlay isKindOfClass:[BMKPolyline class]]){
        BMKPolylineView* polylineView = [[BMKPolylineView alloc] initWithOverlay:overlay];
        polylineView.strokeColor = kBlueColor;
        polylineView.lineWidth = 2.0 * KMINScreenWidthScale;
        return polylineView;
    }
    return nil;
}
```

#### 矩形

```
@property (nonatomic, strong) NSMutableArray *rectangleCoordinateArr;
- (void)addBaiduRectangle
{
    //记得先清空地图,如果有必要的话
    MINCoordinateObject *firstObj = self.rectangleCoordinateArr.firstObject;
    MINCoordinateObject *lastObj = self.rectangleCoordinateArr.lastObject;
    CLLocationCoordinate2D firstCoor = firstObj.coordinate;
    CLLocationCoordinate2D lastCoor = lastObj.coordinate;
    CLLocationCoordinate2D leftTop;
    CLLocationCoordinate2D leftBottom;
    CLLocationCoordinate2D rightTop;
    CLLocationCoordinate2D rightBottom;
    if (firstCoor.latitude > lastCoor.latitude && firstCoor.longitude < lastCoor.longitude) {
        leftTop = firstCoor;
        rightBottom = lastCoor;
        leftBottom.latitude = lastCoor.latitude;
        leftBottom.longitude = firstCoor.longitude;
        rightTop.latitude = firstCoor.latitude;
        rightTop.longitude = lastCoor.longitude;
    }else if (firstCoor.latitude > lastCoor.latitude && firstCoor.longitude > lastCoor.longitude) {
        rightTop = firstCoor;
        leftBottom = lastCoor;
        leftTop.latitude = firstCoor.latitude;
        leftTop.longitude = lastCoor.longitude;
        rightBottom.latitude = lastCoor.latitude;
        rightBottom.longitude = firstCoor.longitude;
    }else if (firstCoor.latitude < lastCoor.latitude && firstCoor.longitude < lastCoor.longitude) {
        leftBottom = firstCoor;
        rightTop = lastCoor;
        leftTop.latitude = lastCoor.latitude;
        leftTop.longitude = firstCoor.longitude;
        rightBottom.latitude = firstCoor.latitude;
        rightBottom.longitude = lastCoor.longitude;
    }else {
        leftTop = lastCoor;
        rightBottom = firstCoor;
        leftBottom.latitude = firstCoor.latitude;
        leftBottom.longitude = lastCoor.longitude;
        rightTop.latitude = lastCoor.latitude;
        rightTop.longitude = firstCoor.longitude;
    }
    CLLocationCoordinate2D coords[4];
    coords[0] = leftTop;
    coords[1] = rightTop;
    coords[2] = rightBottom;
    coords[3] = leftBottom;
    BMKPolygon *polygon = [BMKPolygon polygonWithCoordinates:coords count:4];
    [self.mapView addOverlay:polygon];
}

#pragma mark - BMKMapViewDelegate
- (BMKOverlayView *)mapView:(BMKMapView *)mapView viewForOverlay:(id <BMKOverlay>)overlay{
    if ([overlay isKindOfClass:[BMKPolygon class]]){
        BMKPolygonView* polygonView = [[BMKPolygonView alloc] initWithOverlay:overlay];
        polygonView.strokeColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
        polygonView.fillColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
        return polygonView;
    }
    return nil;
}
```

#### 多边形

```
// CLLocationCoordinate2D是一个结构体，不能保存到数组头
@interface MINCoordinateObject : NSObject
@property (nonatomic, assign) CLLocationCoordinate2D coordinate;
@end
@implementation MINCoordinateObject
@end

@property (nonatomic, strong) NSMutableArray *polygonCoordinateArr;
- (void)addBaiduPolygon
{
    //记得先清空地图,如果有必要的话
    CLLocationCoordinate2D coords[self.polygonCoordinateArr.count];
    for (int i = 0; i < self.polygonCoordinateArr.count; i++) {
        MINCoordinateObject *obj = self.polygonCoordinateArr[i];
        coords[i] = obj.coordinate;
    }
    BMKPolygon *polygon = [BMKPolygon polygonWithCoordinates:coords count: self.polygonCoordinateArr.count];
    [self.mapView addOverlay:polygon];
}

#pragma mark - BMKMapViewDelegate
- (BMKOverlayView *)mapView:(BMKMapView *)mapView viewForOverlay:(id <BMKOverlay>)overlay{
    if ([overlay isKindOfClass:[BMKPolygon class]]){
        BMKPolygonView* polygonView = [[BMKPolygonView alloc] initWithOverlay:overlay];
        polygonView.strokeColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
        polygonView.fillColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
        return polygonView;
    }
    return nil;
}
```

### 地图风格

[可选风格地址](http://lbsyun.baidu.com/custom/) 在个性化模板中选择自己想要的样式，然后点击查看JSON，复制使用

设定文件路径

1、将配置好的样式文件(如：custom_config)放入工程目录任意路径;

2、设定地图样式文件的路径，通过以下方法设定自定义地图样式文件的绝对路径（注意：必须在BMKMapView对象初始化之前设置）

```
//个性化地图模板文件路径
    NSString* path = [[NSBundle mainBundle] pathForResource:@"custom_config" ofType:@""];
//设置个性化地图样式
[BMKMapView customMapStyle:path];
```

切换自定义地图

自v3.0起，支持个性化地图和普通地图切换。v3.3.2起，设置个性化地图后，个性化地图默认为关闭状态，需要设置生效。


```
[BMKMapView enableCustomMapStyle:YES];//打开个性化地图
关闭个性化地图方法如下，

[BMKMapView enableCustomMapStyle:NO];//关闭个性化地图
通过此功能，可实现App的夜间和普通地图切换的需求。
```

### 控件和手势 
#### 指南针 和 My Location 按钮 楼层选择器

**指南针**

Google Maps SDK for iOS 提供的指南针图形在某些情况下会出现在地图的右上角。 仅当摄像头的朝向使得其具有非零方位时，才会出现指南针。 当用户点击指南针时，摄像头以动画呈现方式恢复到方位为零（默认朝向）的位置，并且指南针会在之后不久逐渐消失。

默认情况下，指南针处于禁用状态。 您可以通过将 GMSUISettings 的 compassButton 属性设置为 YES 来启用指南针。 不过，您无法强制始终显示指南针。

```
mapView.settings.compassButton = YES;
```


**My Location 按钮**

仅当启用了 My Location 按钮时，My Location 按钮才会出现在屏幕的右下角。 当用户点击该按钮时，如果当前已知用户的位置，摄像头会以动画方式聚焦于用户的当前位置。

```
mapView.settings.myLocationButton = YES; // 这个是按钮，点击会让屏幕中心移到自己的位置
// 注意与显示当前位置的区别
mapView.myLocationEnabled = YES; // 这个是显示自己的位置图标
```
**楼层选择器**

只要室内地图占有突出位置，楼层选取器控件就会出现在屏幕右下方附近。 当显示两个或更多个室内地图时，楼层选取器将以最靠近屏幕中心的建筑作为参照物。 首次显示选取器时，每栋建筑都有一个默认选定的楼层。

您可以通过选取器来选择其他楼层。

您可以通过将 GMSUISettings 的 indoorPicker 属性设置为 NO 来禁用楼层选取器控件。

```
GMSMapView *mapView = [GMSMapView mapWithFrame:CGRectZero camera:camera];
mapView.settings.indoorPicker = NO;
```

#### 地图手势

**地图平移**

通过调用BMKMapView的 scrollEnabled 属性控制是否启用或禁用平移的功能，默认开启。

```
self.mapView.scrollEnabled = NO;
```

**地图缩放**

缩放手势可改变地图的缩放级别，地图响应的手势如下：

* 双击地图可以使缩放级别增加1 (zoomEnabledWithTap)

* 两个手指捏/拉伸 (zoomEnabled)

通过调用BMKMapView的 zoomEnabled和zoomEnabledWithTap属性控制是否启用或禁用缩放手势，默认开启。如果启用，用户可以双指点击或拉伸地图视图。禁用缩放手势的代码如下：

```
self.mapView.zoomEnabled = NO;
// 开启时，两指单击地图，地图缩小；单指双击地图，地图放大
self.mapView.zoomEnabledWithTap = NO;
```

**地图旋转**

您可以用两个手指在地图上转动。 通过调用BMKMapView的属性rotateEnabled控制是否启用或禁用地图旋转功能，默认开启。如果启用，则用户可使用双指 旋转来旋转地图。

```
self.mapView.rotateEnabled = NO;
/// 地图旋转角度，在手机上当前可使用的范围为－180～180度
@property (nonatomic) int rotation;
```

**禁止所有手势**

通过调用BMKMapView的属性gesturesEnabled控制是否一并禁止所有手势，默认关闭。如果启用，所有手势都将被禁用。

```
self.mapView.gesturesEnabled = NO;
```

### 动画（轨迹回放）

实现思路也只是自定义一个标记，然后通过动画改变标记位置

```
// 自定义BMKAnnotationView，用于显示运动者
@interface SportAnnotationView : BMKAnnotationView

@property (nonatomic, strong) UIImageView *imageView;
@end

@implementation SportAnnotationView

@synthesize imageView = _imageView;

- (id)initWithAnnotation:(id<BMKAnnotation>)annotation reuseIdentifier:(NSString *)reuseIdentifier {
    self = [super initWithAnnotation:annotation reuseIdentifier:reuseIdentifier];
    if (self) {
        [self setBounds:CGRectMake(0.f, 0.f, 64 * KMINScreenWidthScale, 31.5 * KMINScreenWidthScale)];
        _imageView = [[UIImageView alloc] initWithFrame:CGRectMake(0.f, 0.f,  64 * KMINScreenWidthScale, 31.5 * KMINScreenWidthScale)];
        _imageView.image = [UIImage imageNamed:@"playBack"];
        [self addSubview:_imageView];
    }
    return self;
}

@end
```
实现 BMKMapViewDelegate 代理方法 和相关自定义类

```
// 运动结点信息类
@interface BMKSportNode : NSObject

//经纬度
@property (nonatomic, assign) CLLocationCoordinate2D coordinate;
//方向（角度）
@property (nonatomic, assign) CGFloat angle;
//距离
@property (nonatomic, assign) CGFloat distance;
//速度
@property (nonatomic, assign) CGFloat speed;

@end

@implementation BMKSportNode

@end

@interface MainMapViewController ()
{
    SportAnnotationView *sportAnnotationView;
    NSMutableArray *sportNodes;//轨迹点
    NSInteger sportNodeNum;//轨迹点数,总的个数
    NSInteger currentIndex;//当前结点
    NSInteger lastIndex;// 上一个轨迹结点
    BOOL isAnimate;
}
@end

- (void)addRunningAnnotation
{
    BMKSportNode *node = [sportNodes objectAtIndex: 0];
    sportAnnotation = [[BMKPointAnnotation alloc]init];
    sportAnnotation.coordinate = node.coordinate; // 初始点位置
    [self.mapView addAnnotation:sportAnnotation];
}

#pragma mark - BMKMapViewDelegate
- (BMKAnnotationView *)mapView:(BMKMapView *)mapView viewForAnnotation:(id<BMKAnnotation>)annotation
{
    if (annotation == sportAnnotation) {
        sportAnnotationView = [[SportAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:@"sportsAnnotation"];
        sportAnnotationView.draggable = NO;
        BMKSportNode *node = [sportNodes firstObject];
        sportAnnotationView.imageView.transform = CGAffineTransformMakeRotation(node.angle);
        return sportAnnotationView;
    }
    return nil;
}
```
运动代码

```
- (void)running {
    if (currentIndex >= sportNodeNum - 1) {
        isAnimate = NO;
        currentIndex = 0;
        return;
    }
    BMKSportNode *node = [sportNodes objectAtIndex:currentIndex];
    sportAnnotationView.imageView.transform = CGAffineTransformMakeRotation(node.angle);
    if (lastIndex != currentIndex) { // 轨迹跑完之后，重新开始
        BMKSportNode *node = [sportNodes objectAtIndex:currentIndex];
        sportAnnotation.coordinate = node.coordinate;
        lastIndex = currentIndex;
        if (isAnimate == YES) {
            [self running];
        }
    }else {
        CGFloat time = (22 - self.speed)/10.0 * node.distance/node.speed;
        [UIView animateWithDuration:time animations:^{
            if (currentIndex < sportNodeNum-1) {
                currentIndex++;
                BMKSportNode *node = [sportNodes objectAtIndex:currentIndex];
                sportAnnotation.coordinate = node.coordinate;
            }else {
                return ;
            }
        } completion:^(BOOL finished) {
            lastIndex = currentIndex;
            if (isAnimate) {
                [self running];
            }
        }];
    }
}
```






