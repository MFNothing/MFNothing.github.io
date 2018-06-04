---
layout:     post
title:      "Cocoapods私有仓库和公共库的创建和使用"
subtitle:   "Cocoapods"
date:       2018-05-21 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Cocoapods
---

## Cocoapods私有仓库和公共库的创建和使用



在开始创建和使用之前，你需要了解下面的内容（作者自己总结，可能有不对的地方，请见谅）

* repo 代表一个仓库，用来存放 pod 描述文件（.podspec）的地方。
* 通过 **pod repo** 命令我们可以看到我们的本地的所有 repo 仓库。其中 URL 表示是仓库的git地址。如果我们没有私有库，通常只会存在一个叫 master 的 repo 仓库。也可以通过 **cd ~/.cocoapods/repos** 命令，直接到目录查看。

	```
	lymdeMacBook-Pro:~ lym$ pod repo 
	master
	- Type: git (master)
	- URL:  https://github.com/CocoaPods/Specs.git
	- Path: /Users/lym/.cocoapods/repos/master
	1 repo
	```
* pod 描述文件（.podspec）是用来告诉 pod 怎样去生成我们的组件的文件。我们通过配置这个文件来实现私有库的基本配置。（包括库的名字，来源，资源文件目录等，后面会具体介绍怎么写这个文件）
* 我们以往的执行 **pod install** 命令，具体就是从repo库源地址去查找配置文件，如果 repo 库在本地存在，就从本地路径去查找有没有我们要安装的组件，有就安装，没有就报错。一般情况下我们不会在 podfile 文件里面去写 **source 'https://github.com/CocoaPods/Specs.git'** 因为默认会先从本地的 master 这个 repo 库找。如果我们使用私有库就需要去添加这个源，表示先从这个 repo 库地址去查找。
* 所以当你去使用其他第三方库，出现找不到库的情况的时候，就可以通过 **pod repo update** 更新这个本地 repo 库。
* pod 命令中，我们常用 **--help** 来查询有哪些命令可以使用， 使用 **--verbose** 来打印执行过程中的一些信息。例如：**pod trunk me --verbose**, trunk 命令通常会超时，所以打印更多的信息，可以让我们了解，这个命令是否执行成功，卡住很久就可以关掉了。



### 创建私有库

1. 新建一个 git 仓库作为存放 pod 描述文件的库，其他人可以通过这个仓库 Git 地址，向 podfile 中添加 **source '你的存放 pod 描述文件的仓库地址.git'** 来安装你的私有组件库。
2. 通过 **pod repo add [自定义私有库名字] [git地址，我用的码云]** 命令添加你的私有库，执行成功后，可以通过 **pod repo** 命令或者打开 **~/.cocoapods/repos** 目录查看。
```
pod repo add MINRepo https://gitee.com/MFNothing/MINRepo.git
``` 
3. 创建你的组件库。新建一个 git 仓库，这个仓库用来存放你的组件，要保证你的组件可以直接拖入工程能直接运行，或添加依赖库之后就可以运行。也可以直接使用现成的 git 仓库。
```
git clone https://gitee.com/MFNothing/MINTemp.git
```
4. 创建 pod 描述文件（.podspec）。这个文件应该在你的git仓库的一级目录下面。
```
pod spec create MINTemp
```
5. 通过 **tree -L 2** 命令查看文件结构

	```
	lymdeMacBook-Pro:MINTemp lym$ tree -L 2
	.
	├── MINTemp.podspec
	├── README.md
	└── Tool
    	├── MINTool.h
    	└── MINTool.m

	1 directory, 4 files
	```
6. 配置 pod 描述文件。用工具（我用的Sublime Text）打开文件。修改
	
	```
	Pod::Spec.new do |s|
  s.name         = "MINTemp" 
  # 你的组件库名字，也就是你后面在 podfile 文件中填写的 pod 'MINTemp'
  s.version      = "0.0.1" 
  # 版本号，这个看你自己，从低开始定，毕竟后面会更新版本
  s.summary      = "Made by MFNothing" 
  # 概要，随便写
  s.description  = <<-DESC
                  MFNothing's MINTemp
                   DESC
  # s.description 要写，不然会有警告
  s.homepage     = "https://gitee.com/MFNothing" 
  # 写一个能够访问到的地址，一般写组件库存放的地址
  s.license      = "MIT" 
  # 这里要有一个 license 文件，我是从别的库拷贝出来的，github 上面可以创建这样一个文件
  s.author             = "MFNothing" 
  # 作者
  s.source       = { :git => "https://gitee.com/MFNothing/MINTemp.git", :tag => "#{s.version}" } 
  # 这里要将 git 后面对应的地址替换成你组件库的 git 地址
  s.source_files  = "Tool/*" 
  # 资源文件的路径，这个路径是相对于 .podsepc 文件的路径
  s.public_header_files = "Tool/*.h" 
  # s.source_files 公共头文件
  #  s.ios.frameworks  iOS环境下依赖的系统库
end
	```
7. 通过 **pod lib lint** 来校验 pod 描述文件配置对没有。如果正确会这样显示 :

	```
	lymdeMacBook-Pro:MINTemp lym$ pod lib lint
 	-> MINTemp (0.0.1)
	MINTemp passed validation.
	```
8. 提交的你的组件库代码，并打一个 tag，这个 tag 号一定要跟你 pod 描述文件中的 s.version 一致。这样以后你每次修改提交之后，都打一个新的tag，这样就可以生成一个新的版本。（后面会讲怎么删除tag）

	```
	git add ./
	git commit -m "初始化"
	git push
	git tag 0.0.1
	git push --tag
	```
9. 通过 **pod spec lint --sources=https://gitee.com/MFNothing/MINTemp.git** 或者 **pod spec lint** 来校验远程仓库的 pod 描述文件。 pod spec 相对于 pod lib 会更为精确，pod lib 相当于只验证一个本地仓库，pod spec会同时验证本地仓库和远程仓库。
10. 将组件中 pod 描述文件提交给你的 repo 仓库，这样其他人通过在 podfile 中添加 **source 'https://gitee.com/MFNothing/MINRepo.git'** 就可以使用的你的组件了。注意 Podfile 文件中添加的是 repo 仓库地址，不是你的组件的地址。

	```
	pod repo push MINRepo MINTemp.podspec
	```
11. 成功之后，就是测试了，新建一个项目，通过 **pod init** 命令创建一个 podfile 文件，然后编辑它。

	```
	source 'https://gitee.com/MFNothing/MINRepo.git'
	source 'https://github.com/CocoaPods/Specs.git'
	platform :ios, '9.0'
	use_frameworks!
	target 'MINTestPod' do
 	pod 'MINTemp'
 	#, :path => '/Users/lym/Desktop/github/MINTemp/MINTemp.podspec' 
 	# 上面注释的这行，当你创建好 pod 描述文件之后，可以通过在 pod 'MINTemp' 后加上这一行，来测试你的组件库是否能够安装成功。
	end
	```
	
### 创建公共库

1. 执行前面私有库创建的 1 ~ 8 步。
2. 通过 trunk 命令来提交到 cocoapods 仓库中。
3. 通过 **pod trunk me --verbose** 命令来查看自己有没有注册过。如果没有执行下一步，注册后跳过下一步。 这一步经常应该超时报错，所以多试试。

	```
	opening connection to trunk.cocoapods.org:443...
	- Name:     MFNothing
	- Email:    2492505031@qq.com
	- Since:    May 17th, 01:01
	- Pods:
	- MINTool # 这个是作者之前提交成功的，失败了很多次，都是超时。
	- Sessions:
	- May 17th, 01:01 - September 26th, 02:59. IP: 118.112.56.3
	```
4. 通过 **pod trunk register [邮箱地址] [你的用户名]** 来注册，会向你填写的邮箱地址发送一封邮件，点击邮件中的链接之后就算注册成功了。然后再试一下上一步的命令，应该跟作者显示的差不多。
5. 进入你的组件库目录，通过 **pod trunk push MINTemp.podspec --verbose** 将组件库的 pod 描述文件提交到 cocoapods 仓库中。
6. 提交之后，新建一个项目，通过 **pod init** 命令创建一个 podfile 文件，然后编辑它。

	```
	platform :ios, '8.0'
	target 'MINApplicationStart' do
	pod 'MINTestPod'
	end
	```
7. 再提一点，如果想要在其他电脑上也能用上你的库，更新本地 repo 仓库。

	```
	pod repo update
	```
8. 成功之后会显示

	```
	- Data URL: https://raw.githubusercontent.com/CocoaPods/Specs/1d0a4486fd45cf77cbccfcce1f8d65e376e8bd21/Specs/7/b/1/MINTemp/0.0.1/MINTemp.podspec.json
	- Log messages:
	- May 21st, 03:14: Push for `MINTemp 0.0.1' initiated.
	- May 21st, 03:14: Push for `MINTemp 0.0.1' has been pushed (1.806639077 s).
	```
	
### 一些查找资料中遇到的点

#### tag

```
删除本地tags
git tag -d + 分支名称就会删除本地的分支
git tag -d 0.0.1
删除远程分支
git push origin :refs/tags/分支名称 就删除了远程分支
git push origin :refs/tags/0.0.1
```

#### pod 描述文件其他配置，待测试

```
  s.subspec 'Serialization' do |ss|
    ss.source_files = 'AFNetworking/AFURL{Request,Response}Serialization.{h,m}'
    po= 'AFNetworking/AFURL{Request,Response}Serialization.h'
    ss.watchos.frameworks = 'MobileCoreServices', 'CoreGraphics'
    ss.ios.frameworks = 'MobileCoreServices', 'CoreGraphics'
    ss.osx.frameworks = 'CoreServices'
  end
```

*  s.subspec 创建了一个名字 Serialization 的文件夹
*  ss.source_files 指明资源文件位置
*  ss.public_header_files 这个文件夹中的公共头文件
*  ss.watchos.frameworks 手表环境下依赖的系统库
*  ss.ios.frameworks iOS环境下依赖的系统库
*  ss.osx.frameworks   osx 环境下依赖的系统库


```
引用第三方 framework
ss.vendor_frameworks = "thirdSdk.framework"
引用第三方 .a
ss.vendor_libraries = "third.a"
```
	
### 结束

祝好运，pod trunk 命令有毒。后面有时间，作者会再更新，写一些创建新版本组件的内容。
	





