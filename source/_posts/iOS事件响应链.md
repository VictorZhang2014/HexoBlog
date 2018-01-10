---
title: iOS事件响应链
date: 2017-10-10 12:34:50
tags: 事件响应链
categories: iOS
---

# iOS事件是如何响应的？
iOS获取到了用户的“点击”这一行为后，把这个事件封装成UITouch和UIEvent形式的实例，然后找到当前运行的程序，并逐级寻找能够响应这个事件的对象，直到没有响应者响应。这个过程就叫做事件的响应链。

![Events Chain](/img/iOS/events/event_chain.png)

- UITouch是触摸对象
- UIEvent是事件对象

```
//根据坐标返回响应点击的对象
- (nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event;   // recursively calls -pointInside:withEvent:. point is in the receiver's coordinate system

//根据坐标返回事件是否发生在本视图内
- (BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent *)event;   // default returns YES if point is in bounds
```



## 引用
- http://www.cocoachina.com/ios/20160113/14896.html

