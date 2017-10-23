---
title: iOS 常识
date: 2017-10-15 17:03:58
tags: iOS, 架构, 系统的内存分配 
---

## 一、常用架构
**MVC (Model View Controller)** 模型，视图，控制器，模型负责提供数据，视图负责显示，
    控制器的作用就是确保模型和视图的同步，一旦`M改变`，`V`就应该`立即更新`。MVC其实就是一个`环形的形式`
    https://baike.baidu.com/item/MVC框架/9241230?fr=aladdin&fromid=85990&fromtitle=MVC

**MVP (Model View Presenter)** 模型，视图，是从MVC演变而来，它们相通点就是Controller和Presenter
    都是负责处理业务逻辑，但是它们也有很大的区别，就是把`Model`和`View`进行了`分离`，在MVP中，视图并`不`是`直接`使用`模型`，而是通过Presenter来进行的，
    也就是说业务在Presenter内部，数据`获取`和`更新`在`Model内部`，如果视图需要更新，就需要通过Presenter;
    Presenter与View的交互可以是间接的，可以通过接口来更新view，如果view较为复杂，也可以做一个adapter。
    https://baike.baidu.com/item/MVP模式/10961746

**MVVM (Model View View Model)** 一般用在用户控件上，该模式是使用的数据绑定基础架构，
    MVVM是由MVP演变过来的，所以`一些事件`和`命令相关`的东西就放在了`MVVM`中的`VM`，其实也就是相当于`MVP`中的`P`
    https://baike.baidu.com/item/MVVM/96310?fr=aladdin

**ORM (Object Relational Mapping)** 对象关系映射就是一种为了解决面向对象语言与关系型数据库而存在的简易数据的映射，因为是基于对象模型的。
    如iOS CoreData，.NET Entity Framework


## 二、iOS系统的内存分配
1.栈区（stack）由编译器自动分配并释放，先进后出
2.堆区（heap）由程序员分配和释放，如果程序员不释放，在程序也就是该进程结束时，会由操作系统回收
3.全局区（也叫作静态区，static）存放全局变量和静态变量的，未初始化的存在bss段，已初始化的存放在data段
4.常量区 存放常量的，程序结束后由系统释放内存空间
5.代码区 存放程序函数的二进制代码

## 异步绘制
我们进行UITableViewCell重用时，可以把cell的高度进行缓存，以便于下次使用时，直接读取而不用重新计算，计算消耗性能。
异步队列进行绘制UI，使用CoreGraphics框架，其中CoreText是一个文本处理引擎，我们就用这个，它的坐标系统左下角为0,0点


## 离屏渲染：
- 1.当设置CALayer的圆角，alpha，阴影，光栅化，抗锯齿，渐变属性时，发触发离屏渲染机制。
- 2.离屏渲染就是在屏幕以外创建一个内存缓冲区并进行渲染，这很消耗性能
- 3.所以很多时候，我们私用的按钮的圆角，其实是图片背景



<br/>
[GCD的一般认知，打开](/2017/10/15/GCD的一般认知/)
[NSOperation的认知，打开](/2017/10/15/NSOperation的认知/)
[iOS中的锁，打开](/2017/10/15/iOS中的锁/)
[NSThread的认知，打开](/2017/10/15/NSThread的认知/)
[iOS的Runloop认知，打开](/2017/10/15/iOS的Runloop认知/)
