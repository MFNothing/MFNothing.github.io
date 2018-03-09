---
layout:     post
title:      "谷歌地图SDK接入与使用"
subtitle:   "Google Maps SDK"
date:       2018-03-07 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Map
---

## 谷歌地图SDK接入与使用

### 一、接入部分

[官方文档地址](https://developers.google.com/maps/documentation/ios-sdk/start) 

#### cocoapods集成

在 Podfile 中写入下面的代码

```
platform :ios, '8.0'
target '你的项目名字' do
  pod 'GoogleMaps'
end
```

#### 手动安装

1. 下载SDK源文件 [GoogleMaps-2.0.1](https://www.gstatic.com/cpdc/5a212b0fa429156f-GoogleMaps-2.0.1.tar.gz) (可能更新，所以还是去上面的官方文档去下载)
2. 将以下捆绑包拖动到您的项目中（系统提示时，请选择 Copy items if needed）:
	* Subspecs/Base/Frameworks/GoogleMapsBase.framework
	* Subspecs/Maps/Frameworks/GoogleMaps.framework
	* Subspecs/Maps/Frameworks/GoogleMapsCore.framework
3. 在您的项目中右击 GoogleMaps.framework，然后选择 Show In Finder。
4. 从 Resources 文件夹中将 GoogleMaps.bundle 拖动到您的项目中。系统提示时，请确保未选中 Copy items into destination group's folder。
5. 打开 Build Phases 标签，并在 Link Binary with Libraries 中添加以下框架：
	* GoogleMapsBase.framework
	* GoogleMaps.framework
	* GoogleMapsCore.framework
	* GoogleMapsM4B.framework（仅限高级计划客户）
	* Accelerate.framework
	* CoreData.framework
	* CoreGraphics.framework
	* CoreLocation.framework
	* CoreText.framework
	* GLKit.framework
	* ImageIO.framework
	* libc++.tbd
	* libz.tbd
	* OpenGLES.framework
	* QuartzCore.framework
	* SystemConfiguration.framework
	* UIKit.framework
6. 选择您的项目（而不是具体目标），然后打开 Build Settings 标签。在 Other Linker Flags 部分，添加 -ObjC。如果看不到相关设置，请在 Build Settings 栏中将过滤条件由 Basic 更改为 All。

#### 创建API密钥并添加到您的项目中

查找或创建密钥[地址](https://console.developers.google.com/project/_/apiui/credential?authuser=1)

密钥使用，在AppDelegate.m中添加

```
#import <GoogleMaps/GoogleMaps.h>

- (void)setGoogleMaps
{
    [GMSServices provideAPIKey: @"你的密钥"]; 
}
```

### 二、地图使用

#### 地图显示

```
#import <GoogleMaps/GoogleMaps.h>

- (void)initGoogleMap
{
    GMSCameraPosition *camera = [GMSCameraPosition cameraWithLatitude:40.056898
                                                            longitude:116.307626
                                                                 zoom:18];
    GMSMapView *mapView = [GMSMapView mapWithFrame: CGRectMake(0, 0, [[UIScreen mainScreen] bounds].size.width, [[UIScreen mainScreen] bounds].size.height) camera: camera];
    [self.view addSubview: mapView];
}

```

#### 定位

首先需要在 info.plist 添加 NSLocationWhenInUseUsageDescription 或者 NSLocationAlwaysUsageDescription 。根据自己的需求添加。

然后添加以下代码请求用户授权开启定位服务

```
interface ViewController () <CLLocationManagerDelegate>
@property (nonnull, strong) CLLocationManager *locationManager;
@property (nonatomic, strong) GMSMapView *mapView;
@property (nonatomic, assign) CLLocationCoordinate2D myGoogleLocation;
@end

- (void)initLocationManager
{
    //定位管理器
    _locationManager = [[CLLocationManager alloc]init];
    //如果没有授权则请求用户授权
    if ([CLLocationManager authorizationStatus] == kCLAuthorizationStatusNotDetermined){
        [_locationManager requestWhenInUseAuthorization];
    }
    
    if([CLLocationManager authorizationStatus] == kCLAuthorizationStatusAuthorizedWhenInUse || [CLLocationManager authorizationStatus] == kCLAuthorizationStatusAuthorizedAlways){
        if (self.locationManager.delegate == nil) {
            //设置代理
            _locationManager.delegate = self;
            //设置定位精度
            _locationManager.desiredAccuracy=kCLLocationAccuracyBest;
            //定位频率,每隔多少米定位一次
            CLLocationDistance distance = 10.0;//十米定位一次
            _locationManager.distanceFilter = distance;
            [_locationManager setDesiredAccuracy: kCLLocationAccuracyBest];
            //启动跟踪定位
            [_locationManager startUpdatingLocation];
        }
    }
}

#pragma mark - CLLocationManagerDelegate
-(void)locationManager:(CLLocationManager *)manager didUpdateLocations:(nonnull NSArray<CLLocation *> *)locations
{
    [_locationManager stopUpdatingLocation];
    CLLocation *curLocation = [locations firstObject];
    //    通过location  或得到当前位置的经纬度
    CLLocationCoordinate2D curCoordinate2D = curLocation.coordinate;
    self.myGoogleLocation = curCoordinate2D;
    [self showMyLocation];
}

- (void)showMyLocation
{
    dispatch_async(dispatch_get_main_queue(), ^{
        [self.mapView clear];
        NSLog(@"%f, %f", self.myGoogleLocation.latitude, self.myGoogleLocation.longitude);
        self.mapView.camera = [GMSCameraPosition cameraWithLatitude: self.myGoogleLocation.latitude longitude: self.myGoogleLocation.longitude zoom: 16];
    });
}
```

### 显示标记

**创建一个简单的marker**

```
CLLocationCoordinate2D position = CLLocationCoordinate2DMake(10, 10);
GMSMarker *marker = [GMSMarker markerWithPosition:position];
marker.title = @"Hello World"; // 点击会弹出来显示的文字，长度有限制
marker.map = mapView;
```

**删除标记** 

通过将 GMSMarker 的 map 属性设置为 nil，您可以将标记从地图中移除。 或者，您也可以通过调用 GMSMapView 的 clear 方法移除目前地图上的所有叠层（包括标记）。

```
marker.map = nil
// 或者
[mapView clear];
```

**更改标记颜色**

```
marker.icon = [GMSMarker markerImageWithColor:[UIColor blackColor]];
```

**定制标记图像**

如果您希望更改默认标记图像，您可以使用标记的 icon 或 iconView 属性设置自定义图标。

如果设置 iconView，API 将忽略 icon 属性。 只要设置 iconView，就会忽略当前 icon 的更新。

```
marker.icon = [UIImage imageNamed:@"house"];
或者
marker.iconView = [[UIView alloc] initWithFrame: CGRectMake(0, 0, width, height)];
```

**标记平面化**

标记的图标通常根据设备屏幕（而不是地图表面）的方向进行绘制，因此旋转、倾斜或缩放地图不一定会改变标记的朝向。

您可以将标记的朝向设置为平贴地球表面。 平面标记将随地图的旋转而旋转，并在地图倾斜时改变视角。

与常规标记一样，当地图放大或缩小时，平面标记将保持大小不变。

如需更改标记的朝向，请将标记的 flat 属性设置为 YES 或 true。

```
london.flat = YES; // 写轨迹动画的时候就需要扁平化
```

**旋转标记**

默认是0.0，0.0，设置 rotation 属性让标记围绕其锚点旋转。 将旋转指定为 CLLocationDegrees 类型，旋转程度以与默认位置所呈顺时针角度来表示。 当地图上的标记为平面标记时，默认位置为指向北方。

方向是x向右，y向下

```
marker.groundAnchor = CGPointMake(0.5, 0.5); // 标记会看起来左下移动了
CLLocationDegrees degrees = 45;
marker.rotation = degrees;
```
其实也可以通过这个区去设置两个相同坐标的标记的相对位置，当旋转位置变了之后，为了保证旋转点在坐标上，标记会改变它在地图上的位置，所以可以通过设置这个让两个相同坐标并且要全部展示的标记错开。

**信息窗口**

刚刚说的title就是默认的信息窗口，点击marker的时候回出现的东西，还有一个属性snippet，会显示在title的下面，字体要小一点，可以多行，但是也有字数限制。

信息窗口的内容由 title 和 snippet 属性定义。 如果 title 和 snippet 属性均为空白或 nil，点击该标记将不会显示信息窗口。

```
marker.title = @"title one line";
marker.snippet = @"snippet multi-line";
```

**更改信息窗口的位置**

默认偏移为 (0.5, 0.0)，即顶部中心，这里根据的位置都是距离marker位置，比如 (0.0, 1) 表示的旋转点变成了marker的左边底部。

它定义为一个 (x,y) 偏移，其中 x 和 y 的范围都介于 0.0 和 1.0 之间

方向是x向右，y向下

```
marker.infoWindowAnchor = CGPointMake(1, 1);
```
**自定义可点击的信息窗口**

主要是信息窗口你给一个View，它会变成图片，上面的按钮就没了响应。然后可以在self.mapView 上面添加一个自定义的信息窗口，拖动地图的时候也跟到计算信息窗口的位置，改变它的frame

```
@interface ViewController () <GMSMapViewDelegate>
@property (nonatomic, strong) UIView *customInfoView;
@property (nonatomic, strong) GMSMapView *mapView;
@end

- (void)initGoogleMap
{
    GMSCameraPosition *camera = [GMSCameraPosition cameraWithLatitude:40.056898
                                                            longitude:116.307626
                                                                 zoom:18];
    _mapView = [GMSMapView mapWithFrame: CGRectMake(0, 0, [[UIScreen mainScreen] bounds].size.width, [[UIScreen mainScreen] bounds].size.height) camera: camera];
    _mapView.delegate = self; // 设置代理
    [self.view addSubview: _mapView];
}

#pragma mark - GMSMapViewDelegate
- (UIView *)mapView:(GMSMapView *)mapView markerInfoWindow:(GMSMarker *)marker
{
    if ([marker.title isEqualToString: @"marker"]) {
        self.customInfoView = [[UIView alloc] initWithFrame: CGRectMake(0, 0, 100, 50)];
        self.customInfoView.backgroundColor = [UIColor blueColor];
        CLLocationCoordinate2D anchor = [self.mapView.selectedMarker position];
        CGPoint point = [self.mapView.projection pointForCoordinate:anchor];
        point.y -= 100; // 减去marker的高度和他们之间需要的间隙
        self.customInfoView.center = point;
        [self.mapView addSubview: self.customInfoView];
        return [[UIView alloc] initWithFrame: CGRectMake(0, 0, 1, 1)];
    }
    return nil;
}

- (void)mapView:(GMSMapView *)mapView didChangeCameraPosition:(GMSCameraPosition *)position
{
    if (self.mapView.selectedMarker != nil && self.customInfoView.superview) {
        CLLocationCoordinate2D anchor = [self.mapView.selectedMarker position];
        CGPoint point = [self.mapView.projection pointForCoordinate:anchor];
        point.y -= 100;
        self.customInfoView.center = point;
    }else {
        [self.customInfoView removeFromSuperview];
    }
}

- (void)mapView:(GMSMapView *)mapView didTapAtCoordinate:(CLLocationCoordinate2D)coordinate
{
    [self.customInfoView removeFromSuperview];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    if ([keyPath isEqualToString:@"mapView.selectedMarker"]) {
        // 当点击其他marker的时候删除当前的信息窗口
        if (!self.mapView.selectedMarker) {
            [self.customInfoView removeFromSuperview];
        }
    }
}
```
### 图形（围栏）

#### 圆

**fillColor**
用来指定圆的内部颜色的一种 UIColor 对象。 默认为透明。

**strokeColor**
用来指定圆的轮廓颜色的一种 UIColor 对象。 默认值为 blackColor。

**strokeWidth**
圆的轮廓的厚度（以屏幕像素点为单位）。 默认为 1。厚度不会随地图的缩放而缩放。


```
@property (nonatomic, assign) CLLocationCoordinate2D circleCoordinate;

- (void)addGoogleCircleMarkerWithRadius:(NSString *)radius
{
    CGFloat radiusNum = [radius floatValue];
    GMSCircle *circ = [GMSCircle circleWithPosition:self.circleCoordinate
                                             radius:radiusNum];
    circ.fillColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
//    circ.strokeColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
//    circ.strokeWidth = 5;
    circ.map = self.mapView;
}
```

#### 线段

**strokeWidth**
整条线的宽度（以屏幕像素点为单位）。 默认为 1。宽度不会随地图的缩放而缩放。

**geodesic**
如果为 YES，请将此多段线边缘显示为测地线。 测地线段沿地球表面依循最短路径，在采用墨卡托投影法的地图上可能会以曲线形式出现。 非测地线段在地图上绘制为直线。 默认值为 NO。

**spans**
用于指定某条多段线的一个或多个线段的颜色。 spans 属性是 GMSStyleSpan 对象的数组。 设置 spans 属性是更改多段线颜色的首选方法。

**strokeColor**
用来指定多段线颜色的一种 UIColor 对象。 默认值为 blueColor。 如果设置了 spans，则 strokeColor 属性将被忽略。

您可以将多段线从地图上删除，只需将 GMSPolyline 的 map 属性设置为 nil 即可。 或者，您也可以通过调用 GMSMapView clear 方法删除当前地图上的所有叠层（包括多段线和其他形状）。



```
// CLLocationCoordinate2D是一个结构体，不能保存到数组头
@interface MINCoordinateObject : NSObject
@property (nonatomic, assign) CLLocationCoordinate2D coordinate;
@end
@implementation MINCoordinateObject
@end
@property (nonatomic, strong) NSMutableArray *pathleCoordinateArr;
- (void)addGooglePath
{
    //记得先清空地图
    GMSMutablePath *path = [GMSMutablePath path];
    for (int i = 0; i < self.pathleCoordinateArr.count; i++) {
        MINCoordinateObject *obj = self.pathleCoordinateArr[i];
        [path addLatitude: obj.coordinate.latitude longitude: obj.coordinate.longitude];
    }
    GMSPolyline *polyline = [GMSPolyline polylineWithPath:path];
    polyline.strokeWidth = 4 * KMINScreenWidthScale;
    polyline.strokeColor = kBlueColor;
    polyline.geodesic = YES;
    polyline.map = self.mapView;
}
```

#### 矩形

```
@property (nonatomic, strong) NSMutableArray *rectangleCoordinateArr;
- (void)addGoogleRectangle
{
//记得先清空地图
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
    GMSMutablePath *rect = [GMSMutablePath path];
    [rect addCoordinate: leftTop];
    [rect addCoordinate: rightTop];
    [rect addCoordinate: rightBottom];
    [rect addCoordinate: leftBottom];
    
    GMSPolygon *polygon = [GMSPolygon polygonWithPath:rect];
    polygon.fillColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
    polygon.strokeColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
    polygon.strokeWidth = 2;
    polygon.map = _googleMapView;
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
- (void)addGooglePolygon
{
	//记得先清空地图
    GMSMutablePath *path = [GMSMutablePath path];
    for (int i = 0; i < self.polygonCoordinateArr.count; i++) {
        MINCoordinateObject *obj = self.polygonCoordinateArr[i];
        [path addLatitude: obj.coordinate.latitude longitude: obj.coordinate.longitude];
    }
    GMSPolygon *polygon = [GMSPolygon polygonWithPath: path];
    polygon.fillColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
    polygon.strokeColor = [UIColor colorWithRed:0 green:0 blue:0 alpha:0.3];
    polygon.strokeWidth = 2;
    polygon.map = self.mapView;
}
```

### 地图风格

地图显示灰色的风格，要向地图应用自定义地图样式，请调用 GMSMapStyle(...) 来创建一个 GMSMapStyle 实例，传入本地 JSON 文件的网址或者包含样式定义的 JSON 字符串。

[可选风格地址](https://mapstyle.withgoogle.com/)

```
- (void)setMapStyle
{
	self.mapView.mapStyle = [GMSMapStyle styleWithJSONString:@"[{\"featureType\":\"water\",\"elementType\":\"geometry\",\"stylers\":[{\"color\":\"#e9e9e9\"},{\"lightness\":17}]},{\"featureType\":\"landscape\",\"elementType\":\"geometry\",\"stylers\":[{\"color\":\"#f5f5f5\"},{\"lightness\":20}]},{\"featureType\":\"road.highway\",\"elementType\":\"geometry.fill\",\"stylers\":[{\"color\":\"#ffffff\"},{\"lightness\":17}]},{\"featureType\":\"road.highway\",\"elementType\":\"geometry.stroke\",\"stylers\":[{\"color\":\"#ffffff\"},{\"lightness\":29},{\"weight\":0.2}]},{\"featureType\":\"road.arterial\",\"elementType\":\"geometry\",\"stylers\":[{\"color\":\"#ffffff\"},{\"lightness\":18}]},{\"featureType\":\"road.local\",\"elementType\":\"geometry\",\"stylers\":[{\"color\":\"#ffffff\"},{\"lightness\":16}]},{\"featureType\":\"poi\",\"elementType\":\"geometry\",\"stylers\":[{\"color\":\"#f5f5f5\"},{\"lightness\":21}]},{\"featureType\":\"poi.park\",\"elementType\":\"geometry\",\"stylers\":[{\"color\":\"#dedede\"},{\"lightness\":21}]},{\"elementType\":\"labels.text.stroke\",\"stylers\":[{\"visibility\":\"on\"},{\"color\":\"#ffffff\"},{\"lightness\":16}]},{\"elementType\":\"labels.text.fill\",\"stylers\":[{\"saturation\":36},{\"color\":\"#333333\"},{\"lightness\":40}]},{\"elementType\":\"labels.icon\",\"stylers\":[{\"visibility\":\"off\"}]},{\"featureType\":\"transit\",\"elementType\":\"geometry\",\"stylers\":[{\"color\":\"#f2f2f2\"},{\"lightness\":19}]},{\"featureType\":\"administrative\",\"elementType\":\"geometry.fill\",\"stylers\":[{\"color\":\"#fefefe\"},{\"lightness\":20}]},{\"featureType\":\"administrative\",\"elementType\":\"geometry.stroke\",\"stylers\":[{\"color\":\"#fefefe\"},{\"lightness\":17},{\"weight\":1.2}]}]" error: &error];
}
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


您可以通过设置 GMSUISettings 类的属性来禁用地图上的默认手势，该类是 GMSMapView 的一个属性。

可以通过编程方式启用和禁用以下手势。 请注意，禁用手势不会限制以编程方式访问摄像头设置。

scrollGestures 

* 控制是启用还是禁用滚动手势。 如果处于启用状态，用户可以通过滑动来平移摄像头。

zoomGestures 

* 控制是启用还是禁用缩放手势。 如果处于启用状态，用户可以通过双击、双指点按或捏合来缩放摄像头。 请注意，当 scrollGestures 处于启用状态时，双击或捏合可能会使摄像头平移到指定的点。

tiltGestures 

* 控制是启用还是禁用倾斜手势。 如果处于启用状态，用户可以使用两根手指垂直向下或向上滑动来使摄像头倾斜。

rotateGestures 

* 控制是启用还是禁用旋转手势。 如果处于启用状态，用户可以使用两根手指旋转手势来旋转摄像头。

```
mapView.settings.scrollGestures = NO;
mapView.settings.zoomGestures = NO;
```

### 动画（轨迹回放）


```
- (void)animateToNextCoord:(GMSMarker *)marker {
			CoordsList *coords = marker.userData; // @property(nonatomic, strong) id GMS_NULLABLE_PTR userData; 随便自定义的类型都可以
			// 下一个点的位置
    		CLLocationCoordinate2D coord = [coords next]; 
    		// 当前的位置
    		CLLocationCoordinate2D previous = marker.position;
    		// 角度
			CLLocationDirection heading = GMSGeometryHeading(previous, coord);
			// 距离
            CLLocationDistance distance = GMSGeometryDistance(previous, coord);
            if (marker.flat) {
                marker.rotation = heading;
            }
            // Use CATransaction to set a custom duration for this animation. By default, changes to the
            // position are already animated, but with a very short default duration. When the animation is
            // complete, trigger another animation step.
            [CATransaction begin];
            // 计算速度
            [CATransaction setAnimationDuration:(distance / (22 + self.speed * 2))];
            __weak ViewConroller *weakSelf = self;
            [CATransaction setCompletionBlock:^{
                if (isAnimate == YES) { // 判断是否还可以继续执行
                    [weakSelf animateToNextCoord:marker];
                }
            }];
            marker.position = coord;
            [CATransaction commit];
}
```

```
@interface CoordsList : NSObject
@property(nonatomic, readonly, copy) GMSPath *path;
@property(nonatomic, assign) NSUInteger target;

- (id)initWithPath:(GMSPath *)path;

- (CLLocationCoordinate2D)next;

@end

@implementation CoordsList

- (id)initWithPath:(GMSPath *)path {
    if ((self = [super init])) {
        _path = [path copy];
        _target = 0;
    }
    return self;
}

- (CLLocationCoordinate2D)next {
    ++_target;
    if (_target == _path.count) {
        _target = 0;
        CLLocationCoordinate2D coor;
        coor.latitude = -1;
        coor.longitude = -1;
        return coor;
    }
    return [_path coordinateAtIndex:_target];
}

```





