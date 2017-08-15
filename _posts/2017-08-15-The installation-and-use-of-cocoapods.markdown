---
layout:     post
title:      "cocoapods 的安装与使用"
subtitle:   "The installation and use of cocoapod"
date:       2017-08-15 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - cocoapods
---

## 安装（前提是你安装了gem）

* 查看的你的Ruby源,看是不是默认的，如果是默认的需要替换，默认访问速度太慢了。

```
	gem sources -l
```

* 去除默认Ruby源。

```
	gem sources --remove https://rubygems.org/
```

* 添加需要使用的源。

```
	gem sources -a https://ruby.taobao.org/
```

* 安装cocoapods，会输入密码。

```
	sudo gem install cocoapods
```

## 使用

* 进入工程根目录（就是有.xcodeproj文件的目录下），使用vim创建一个Podfile文件，并进入编辑模式（终端输入下面指令就可以了）。

```
	vim Podfile
```

* vim中输入的内容，需要注意的是不输入版本，默认为最新。

```
platform :ios, ‘8.0’
target ‘cocospod-test’ do
	pod 'AFNetworking','~>2.0'
	pod 'SDWebImage'
	pod 'MJRefresh'
end
```

* 安装依赖库。

```
	pod install
```

* 当你修改了Podfile中的内容后，你就可以使用这个命令。

```
	pod update
```
