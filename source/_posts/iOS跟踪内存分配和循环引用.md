---
title: iOS跟踪内存分配和循环引用
date: 2017-10-27 20:27:16
tags: iOS, FBMemoryProfiler
categories: iOS
---

## 概述
我们在开发iOS应用期间，可能有时候就会遇到循环引用，或者内存分配状态等信息的问题，那么有没有办法可以在应用运行期间检测到了？答案是：当然有了！那就是开源的 <a href="https://github.com/facebook/FBMemoryProfiler" target="_blank">FBMemoryProfiler</a> ，由Facebook提供。使用起来也是非常的简单，下面我就讲解下

## FBMemoryProfiler
<a href="https://github.com/facebook/FBMemoryProfiler" target="_blank">FBMemoryProfiler</a>是一个在应用运行期间也可以浏览内存使用情况，专为iOS开发者设计的，基本包含<a href="https://github.com/facebook/FBAllocationTracker" target="_blank">FBAllocationTracker</a>和<a href="https://github.com/facebook/FBRetainCycleDetector" target="_blank">FBRetainCycleDetector</a>。

`FBAllocationTracker`是收集对象的信息，也支持生成和检测循环引用。

<br/>
## 一。FBMemoryProfiler的安装与使用
跟踪与现实内存使用情况

## 1.1 安装
使用`Carthage`，那么你需要在项目的`Cartfile`文件里添加上代码如下：
```
github "facebook/FBMemoryProfiler"
```
`FBMemoryProfiler`需要在非debug（non-debug）模式下编译，所以你需要键入如下命令：
```
carthage update --configuration Debug
```

使用`CocoaPods`，那么需要你在`Podfile`文件里添加如下代码：
```
pod 'FBMemoryProfiler'
```
然后，在`Terminal`里`cd`到项目目录，最后键入`pod install`
这个你完全可以在`Debug`模式下编译，这是有该[编译标签](https://github.com/facebook/FBMemoryProfiler/blob/master/FBMemoryProfiler/FBMemoryProfiler.h#L29)控制的。

## 1.2 使用
1.在`main.m`文件里添加如下代码，意味着开启`FBAllocationTracker`
```
#import <UIKit/UIKit.h>
#import "AppDelegate.h"
#import <FBAllocationTracker/FBAllocationTrackerManager.h>

int main(int argc, char * argv[]) {
    [[FBAllocationTrackerManager sharedManager] startTrackingAllocations];
    [[FBAllocationTrackerManager sharedManager] enableGenerations];
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

2.开启内存占用剖析检测功能
```
#import <FBMemoryProfiler/FBMemoryProfiler.h>

FBMemoryProfiler *memoryProfiler = [FBMemoryProfiler new];
[memoryProfiler enable];

_memoryProfiler = memoryProfiler;
```
`FBMemoryProfiler`一般是你需要在哪个控制器检测，你就把上面这块代码放到哪个控制器；我的建议是放到所有的控制器的父控制器（父控制器是你自定义的类，你所有使用带控制器的类就会从这个类继承），这比较方便，所有的控制器都可以检测到。
运行你的程序后，你就可以看到界面上有一个按钮，点击按钮，你就能看到内存使用情况了。

来看下效果
<img src="/img/iOS/memory/FBMemoryProfiler_Example.gif" alt="" width="320" height="640" />


<br/>

## 二。FBAllocationTracker的安装与使用
跟踪Objective-C对象的分配状态

## 2.1 安装
使用`Carthage`，那么你需要在项目的`Cartfile`文件里添加上代码如下：
```
github "facebook/FBAllocationTracker"
```
`FBAllocationTracker`需要在非debug（non-debug）模式下编译，所以你需要键入如下命令：
```
carthage update --configuration Debug
```

使用`CocoaPods`，那么需要你在`Podfile`文件里添加如下代码：
```
pod 'FBAllocationTracker'
```
然后，在`Terminal`里`cd`到项目目录，最后键入`pod install`
这个你完全可以在`Debug`模式下编译，这是有该[编译标签](https://github.com/facebook/FBAllocationTracker/blob/master/FBAllocationTracker/FBAllocationTrackerImpl.h#L17)控制的。

**这个其实在安装FBMemoryProfiler的时候就安装好了**

## 2.2 使用
其实如果你在第一步已经配置了如下代码，那么这一步就不需要在添加了
```
#import <FBAllocationTracker/FBAllocationTrackerManager.h>

int main(int argc, char * argv[]) {
  [[FBAllocationTrackerManager sharedManager] startTrackingAllocations];
  [[FBAllocationTrackerManager sharedManager] enableGenerations];
  @autoreleasepool {
      return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
  }
}
```
现在我来解释下`FBAllocationTracker`的两种模式：分别是`tracking objects`和`counting allocs/deallocs`，也就是`跟踪对象`和`分配与释放计数`。`tracking objects`比较有意思但是这里不做解释了。`counting allocs/deallocs`意思是当你想使用这个功能并且统计，而且不想影响性能时，你就可以使用这个模式。
`startTrackingAllocations`方法负责替换`NSObject's`的`+alloc`和`-dealloc`方法，当启用`enableGenerations`方法时，会开始跟踪实际对象实例数。

当然，我们也可以来抓取一下该应用的所有的类的分配情况：
```
NSArray<FBAllocationTrackerSummary *> *summaries = [[FBAllocationTrackerManager sharedManager] currentAllocationSummary];
```
`FBAllocationTrackerSummary`会告诉你，在你指定的类里，还有多少个存活的实例对象。

当我们指定一个类，并且要得知该类有多少个活着的实例对象时，可以使用如下代码
```
NSArray *instances =[[FBAllocationTrackerManager sharedManager] instancesOfClasses:@[[ViewController class]]];
```

我们也可以查看<a href="https://github.com/facebook/FBAllocationTracker/blob/master/FBAllocationTracker/FBAllocationTrackerManager.h" target="_blank">FBAllocationTrackerManager</a>头文件来得知更多的功能。


<br/>

## 三。FBRetainCycleDetector的安装与使用
<a href="https://github.com/facebook/FBRetainCycleDetector" target="_blank">FBRetainCycleDetector</a>是用来检测在iOS应用运行期间出现的循环引用问题。循环应用(Retain Cycles)是比较经典的问题，因为它会引起内存泄露。随着业务的增加，代码的复杂度也随着增加了，那么有时候要从代码中找出来哪一行代码引起了循环引用，这是一个头疼的问题，但是了？<a href="https://github.com/facebook/FBRetainCycleDetector" target="_blank">FBRetainCycleDetector</a>就是来解决这个问题的。

## 3.1 安装
安装很简单，跟上面的一样，如果安装了`FBMemoryProfiler`，那它会把这三个库一并安装上。

## 3.2 使用
### 3.3.1 检测循环引用
比较简单
```
//导入头文件
#import <FBRetainCycleDetector/FBRetainCycleDetector.h>

FBRetainCycleDetector *detector = [FBRetainCycleDetector new];
[detector addCandidate:myObject];
NSSet *retainCycles = [detector findRetainCycles];
NSLog(@"%@", retainCycles);
```
`myObject`就是你想要跟踪的实例对象。
`retainCycles`就是返回循环引用对象的个数，及每一条循环应用的路径

### 3.3.2 过滤循环引用对象
过滤掉你不想检测到的循环引用对象，因为并不是每个循环应用都是内存泄露，代码如下
```
NSMutableArray *filters = @[
  FBFilterBlockWithObjectIvarRelation([UIView class], @"_subviewCache"),
];

FBObjectGraphConfiguration *configuration =
[[FBObjectGraphConfiguration alloc] initWithFilterBlocks:filters
                                     shouldInspectTimers:YES];
FBRetainCycleDetector *detector = [[FBRetainCycleDetector alloc] initWithConfiguration:configuration];
[detector addCandidate:myObject];
NSSet *retainCycles = [detector findRetainCycles];
NSLog(@"%@", retainCycles);
```
每一个过滤器其实是一个block，每个block有两个`FBObjectiveCGraphElement`对象，可以说他们的关系是有效的。

### 3.3.3 objc_setAssociatedObject
Objective-C允许我们对类进行动态的添加成员变量，可以通过`objc_setAssociatedObject`方法。那么当我们使用`OBJC_ASSOCIATION_RETAIN_NONATOMIC`策略时，有可能这些方法也会引起循环引用。`FBRetainCycleDetector`也可以捕获到这样的循环引用，但是，我们需要多一步设置，代码如下：
```
#import <FBRetainCycleDetector/FBAssociationManager.h>

int main(int argc, char * argv[]) {
  @autoreleasepool {
    [FBAssociationManager hook];
    return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
  }
}
```
上面这块代码`[FBAssociationManager hook]`其实使用的是<a href="https://github.com/facebook/fishhook" target="_blank">fishhook</a>，因为它可以干涉函数`objc_setAssociatedObject`和`objc_resetAssociatedObjects`，并且跟踪它们的引用


好了，就这么多了。。。

<br/>
<br/>

## 相关链接
<a href="https://github.com/facebook/FBMemoryProfiler" target="_blank">FBMemoryProfiler</a>
<a href="https://github.com/facebook/FBAllocationTracker" target="_blank">FBAllocationTracker</a>
<a href="https://github.com/facebook/FBRetainCycleDetector" target="_blank">FBRetainCycleDetector</a>
<a href="https://github.com/facebook/fishhook" target="_blank">fishhook</a>
