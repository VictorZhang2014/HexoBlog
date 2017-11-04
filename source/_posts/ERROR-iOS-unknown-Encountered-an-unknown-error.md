---
title: 'ERROR | [iOS] unknown: Encountered an unknown error '
date: 2016-10-20 18:22:38
tags: cocoapods
categories: iOS
---

## 使用环境
Xcode 8.0 正式版
cocoapods 1.0.1

#### 运行命令
```
> pod spec lint ZRQRCodeViewController.podspec
```

提交本人的ZRQRCodeViewController到cocoapods中心时，报错ERROR | [iOS] unknown: Encountered an unknown error  xxxxxxxxx

```
> 原因：
> 1.XCode 8.0环境下，要求cocoapods 1.1.0
> 2.XCode 8.0必须下载iPhone Simulator 9.3的模拟器
```

如果不安装这两个，会发生一些意想不到问题，建议安装和更新，虽然模拟器很慢，本人也是安装了好几天才把iPhone Simulator 9.3安装上，香菇蓝瘦~~~

读者请阅读这里 https://github.com/CocoaPods/CocoaPods/issues/5702

## 解决方案：
#### 1.更新cocoapods的版本到1.1.0
```
> //查看当前从哪里下载的cocopods
> gem sources -l
>
> //如果不是https://ruby.taobao.org/，就设置下载地址
> gem sources —add https://ruby.taobao.org/
>
> //安装cocoapods
> sudo gem install cocoapods

> //安装完了之后
> pod setup
```

#### 2.下载iPhone Simulator 9.3 ，这个就不用说了吧，直接去xcode 8.0的Preferences里找到Components菜单项，找到9.3的模拟器下载就行

