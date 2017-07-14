---
layout:     post
title:      "jekyll 安装与使用"
subtitle:   "Installation and use of jekyll"
date:       2017-07-14 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - jekyll
    - git
---

## jekyll 的安装

在终端中一次输入

1. 使用curl -l 进行重定向
```
curl -L https://get.rvm.io | bash -s stable
```
2. 了解现在rvm版本
```
rvm list known
```
3. 安装最新版本
```
rvm install 2.4
```
4. 查看安装的的版本
```
rvm list
```
5. 设置default
```
rvm 2.4.0 --default
```
6. 删除之前安装的低版本
```
rvm remove 2.0.0
```
7. 查看rvm 安装情况
```
rvm -v
```
8. 查看 ruby 和 gem 的版本 （jekyll 会要求ruby版本大于2.1）
```
ruby -v
gem -v
```
9. 安装jekyll
```
gem install jekyll
```
10. 查看jekyll 是否安装成功
```
jekyll -v
```

### 其中会遇到的问题

#### 错误1 jekyll-paginate
**Dependency Error:**Yikes! It looks like you don’t have jekyll-paginate or one of its dependencies installed. In order to use Jekyll as currently configured, you’ll need to install this gem. The full error message from Ruby is: ‘cannot load such file – jekyll-paginate’ If you run into trouble, you can find helpful resources at Getting Help
jekyll 3.1.2 | Error: jekyll-paginate

**解决办法:**安装jekyll时候直接运行
```
gem install jekyll-paginate即可解决。
```
#### 错误2 Jekyll::Converters::Markdown encountered an error while conv
**Dependency Error:**Yikes! It looks like you don't have redcarpet or one of its
dependencies installed. 

 In order to use Jekyll as currently configured, you'll n
eed to install this gem. The full error message from Ruby is: 'cannot load such
file -- redcarpet' 

If you run into trouble, you can find helpful resources at http://jekyllrb.com/help/!
  Conversion error: 
  
  Jekyll::Converters::Markdown encountered an error while conv
erting '_posts/2014-12-28-first-blog.md':
redcarpet

ERROR: YOUR SITE COULD NOT BE BUILT: redcarpet

**解决办法:** _config.yml中的markdown改为kramdown


#### 错误3 did not find expected key while parsing a block mapping at line 2 column 1
**jekyll Error:**(/Users/lewei/Desktop/git blog/huxpro.github.io-master/_config.yml): did not find expected key while parsing a block mapping at line 2 column 1

**中文** 缩进的问题，第一行title之前需要一个空格

\*title: MFNothing Blog （\*号表示空格）

**英文** Indentation is very important in Liquid, and you have to make sure you have the right amount of spaces when declaring collections.
In your case, you have an extra space before - title: bla which is causing the error.
i.e. you have:

```
 - title: bla
```
 
when you should have:

```

- title: bla
```