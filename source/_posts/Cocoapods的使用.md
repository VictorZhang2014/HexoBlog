---
title: Cocoapods的使用
date: 2017-04-01 16:27:50
tags: CocoaPods
categories: iOS
---

CocoaPods Command 

## 使用Cocoapods
安装Cocoapods前以前要更新 Ruby到2.3以上版本，否则各种报错

异常：如果下载不下来，就拷贝别人电脑，可以下载的那个

升级Ruby最新版
```
sudo gem update —system
```

查看当前从哪里下载的cocopods
```
gem sources -l
```

```
gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
```

移除下载地址
```
gem sources —remove https://rubygems.org/
```

设置下载地址
```
gem sources —add https://ruby.taobao.org/
```

添加你找到的可用的镜像源
```
gem sources -a http://rubygems-china.oss.aliyuncs.com
```

安装cocoapods
```
sudo gem install cocoapods
```
会提示输入密码
```
pod setup
```



> 1.定位到项目目录
  ```
  cd 
  ```
> 2.初始化Pod
  ```
  pod init
  ```
> 3.编辑该文件
   ```
   vi Podfile
   ```
   内容是
   ```
   platform :ios, '7.0'

   use_frameworks! //添加了这句就必须使用iOS8.0

target ‘WeiMeiBrowser’ do
   pod 'SDWebImage', '~> 3.7.1' 
   pod 'Toast', '~> 2.4' 
   pod 'SVProgressHUD', '~> 1.1.2'
   pod 'M80AttributedLabel', '~> 1.3.1'
   pod 'FMDB', '~> 2.5'
   pod 'Reachability', '~> 3.1.1'
   pod 'CocoaLumberjack', '~> 2.0.0-rc2'
end   
```

   :wq保存退出

> 4.安装
    ```
    pod install

    Pod install —verbose —no-repo-update

    Pod update —verbose —no-repo-update
    ```







## 以下是发布开源框架到Cocoapods中心服务器

> 1.在目录下执行
```
pod spec create myname.podspec
```
执行后会生成podname.podspec文件，修改文件内容如官网描述
http://guides.cocoapods.org/syntax/podspec.html

```
Pod::Spec.new do |spec|
    spec.name         = 'ZRAlertController'
    spec.version      = '1.0'
    spec.license      = "MIT"
    spec.homepage     = 'https://github.com/VictorZhang2014/ZRAlertController'
    spec.author       = { "Victor Zhang" => "victorzhangq@gmail.com" }
    spec.summary      = 'UIAlertController provides alert view functions.'
    spec.source       = { :git => 'https://github.com/VictorZhang2014/ZRAlertController.git', :tag => spec.version.to_s }
    spec.platform = :ios
    spec.source_files = 'Classes/ZRAlertController.{h,m}'
    spec.framework    = 'UIKit'
    spec.requires_arc = true
end
```

> 2.添加tag，并且提交到远程
```
git tag 1.0
git push —tags
git tag -d 2.0    删除tag
```

> 3.检测配置是否正确
```
pod spec lint ZRAlertController.podspec
```

> 4.检测无误后，注册一个Cocopods的session（如果已注册，可以略过）
这是官网介绍
http://guides.cocoapods.org/making/getting-setup-with-trunk.html

```
pod trunk register victorzhangq@gmail.com 'Victor Zhang' 
--description='macbook pro'
```
这个description可以不填写
在邮箱收到确认email后，它会提示你点击确认

> 5.可以提交项目到Cocopods了
```
pod trunk push ZRAlertController.podspec 
```

> 6.可以通过pod search来搜索cocoapods库
```
pod search ZRAlertController
```





