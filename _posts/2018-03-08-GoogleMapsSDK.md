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















