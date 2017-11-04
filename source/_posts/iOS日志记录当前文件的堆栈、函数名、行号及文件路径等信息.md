---
title: iOS日志记录当前文件的堆栈、函数名、行号及文件路径等信息
date: 2017-10-30 21:51:49
tags: iOS, 日志
categories: iOS
---

iOS日志记录当前文件的堆栈、类名、函数名、行号及文件路径等信息
```
        NSArray *array = [NSThread callStackSymbols];
        NSLog(@"堆栈信息：        %@", array);
        NSLog(@"当前类名：         %@", NSStringFromClass([self class]));
        NSLog(@"当前函数名：       %s", __func__);
        NSLog(@"当前函数和参数：    %s", __PRETTY_FUNCTION__);
        NSLog(@"当前函数的行号：    %d", __LINE__);
        NSLog(@"当前文件路径：      %s", __FILE__);
```
输出结果为：
```
2017-10-30 18:58:31.740513+0800 TestProject[8383:291415] 堆栈信息：        (
	0   TestProperties                      0x000000010aa633da -[ViewController viewDidLoad] + 74
	1   UIKit                               0x000000010e3e254d -[UIViewController loadViewIfRequired] + 1235
	2   UIKit                               0x000000010e3e299a -[UIViewController view] + 27
	3   UIKit                               0x000000010e2b0ae3 -[UIWindow addRootViewControllerViewIfPossible] + 122
	4   UIKit                               0x000000010e2b11eb -[UIWindow _setHidden:forced:] + 294
	5   UIKit                               0x000000010e2c4098 -[UIWindow makeKeyAndVisible] + 42
	6   UIKit                               0x000000010e236521 -[UIApplication _callInitializationDelegatesForMainScene:transitionContext:] + 4711
	7   UIKit                               0x000000010e23b751 -[UIApplication _runWithMainScene:transitionContext:completion:] + 1720
	8   UIKit                               0x000000010e600e00 __111-[__UICanvasLifecycleMonitor_Compatability _scheduleFirstCom
2017-10-30 18:58:31.740513+0800 TestProject[8383:291415] 当前类名：         ViewController
2017-10-30 18:58:31.740513+0800 TestProject[8383:291415] 当前函数名：       -[ViewController viewDidLoad]
2017-10-30 18:58:31.740513+0800 TestProject[8383:291415] 当前函数和参数：    -[ViewController viewDidLoad]
2017-10-30 18:58:31.740513+0800 TestProject[8383:291415] 当前函数的行号：    28
2017-10-30 18:58:31.740513+0800 TestProject[8383:291415] 当前文件路径：      /Users/Victor/Documents/Repositories/TestProject/TestProject/ViewController.m
```

**适用于C++/C/Objective-C**


