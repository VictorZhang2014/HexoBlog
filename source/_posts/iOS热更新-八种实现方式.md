---
title: iOS热更新 - 八种实现方式
date: 2017-10-29 12:02:22
tags: iOS, HotPatch
categories: iOS
---

## 一、JSPatch
个人开发者开源的<a href="https://github.com/bang590/JSPatch" target="_blank">JSPatch</a>，做热更新时，从服务器拉去js脚本。理论上可以修改和新建所有的模块，但是不建议这样做。
建议 用来做紧急的小需求和 修复严重的线上bug。


## 二、lua脚本
阿里开源的<a href="https://github.com/alibaba/wax" target="_blank">wax</a>，热更新时，从服务器拉去lua脚本。游戏开发经常用到。也可以用于iOS应用的热更新/修复。


## 三、Weex
阿里开源的<a href="https://github.com/apache/incubator-weex" target="_blank">weex</a>跨平台，一套代码，iOS、Android都可以运行。用前端语法实现原生效果。比React Native更好用。
weex基于vue.js，ReactNative使用React。
ReactNative安装配置麻烦。 weex安装cli之后就可以使用。

react模板JSX有一定的学习成本，vue和常用的web开发类似，模板是普通的html，数据绑定用mustache风格，样式直接使用css。

淘宝干的漂亮，中国在编码的实力越来越牛叉了。威武！！！ 


## 四、React Native
Facebook开源的<a href="https://github.com/facebook/react-native" target="_blank">React Native</a>
不像Weex能一套代码多端运行，需要自己分别做修改。

React Native 可以动态添加业务模块，但无法做到修改原生OC代码。

JSPatch、lua 配合React Native可以让一个原生APP时刻处于可扩展可修改的状态。

 

## 五、Hybrid

像PhoneGap之类的框架, 基本概念和web差不多, 通过更新js/html来实现动态化，没有原生的效果流畅。

 

## 六、动态库
可以做demo用，真实使用的时候会被苹果禁止。

因为 打包发到AppStore的ipa安装包 里的每个动态库 都有唯一的编码，iOS系统会进行验证，所以动态通过网络获取 新的动态库 也用不了。

 

## 七、rollout.io

<a href="https://rollout.io/" target="_blank">rollout</a>紧急修复线上bug。后端有相关的管理页面。因为是国外的网站，然后呢，要翻墙才能使用。

 

## 八、DynamicCocoa
滴滴iOS的一个框架，准备在2017年初开源，与JSPatch比更加智能化，用OC在XCode中写完代码，用工具可以自动生成可以更新的js文件。
到现为止也用不了。

我猜微信和QQ都自己研发了一套热更新的代码，知道原理了，其实自己搞一套也不是很难。

 
