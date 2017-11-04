---
title: You should know the easy-to-use objc_msgSend
date: 2017-10-10 12:15:12
tags: objc_msgSend, runtime
categories: iOS
---

# Too many arguments to function call, expected 0, have 2

When I created an new iOS project, I got this error as shown below.
<img src="/img/iOS/runtime/iOSRuntime_objc_msgSend_got_error.jpeg" alt="" />

But why I got this error? It's easy to know because of Enabling Strict Checking of objc_msgSend Calls in default mode.

<br/>

# To solve it
Select your project -> select your target  ->  Build Settings  -> Search by keyword "objc_msgSend", only one item be shown  ->  toggle it to NO
<img src="/img/iOS/runtime/iOSRuntime_objc_msgSend.jpeg" alt="" />

<br/>
# To make use of objc_msgSend for message forwarding
```
#import "ViewController.h"

#import <objc/message.h>

@interface ViewController ()

@end

@implementation ViewController

//1.无参无返回值
- (void)run1 {
    NSLog(@"1.无参无返回值");
}

//2.有参无返回值
- (void)run2:(NSArray *)parameters {
    NSLog(@"2.有参无返回值 name=%@ age=%d", parameters[0], [parameters[1] intValue]);
}

//3.无参有返回值
- (NSString *)run3 {
    return @"无参有返回值";
}

//4.有参有返回值
- (int)run4:(NSString *)name age:(int)age height:(float)height {
    NSLog(@"4.有参有返回值 name=%@, age=%d, height=%lf", name, age, height);
    return age + height;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    //第一种方式，需要关闭objc_msgSend严格检查，步骤： Build Settings -> 搜索“objc_msgsend” -> 设置为NO
    
    //1.无参无返回值
    objc_msgSend(self, @selector(run1));
    
    //2.有参无返回值
    objc_msgSend(self, @selector(run2:), @[ @"asdf", @12 ]);
    
    //3.无参有返回值
    id returnVal3 = objc_msgSend(self, @selector(run3));
    NSLog(@"3.收到参数: %@", returnVal3);
    
    //4.有参有返回值
    int returnVal4 = (int)objc_msgSend(self, @selector(run4:age:height:), @"qwerty", @22, @60.8);
    NSLog(@"收到参数 %d", returnVal4);
    
    
    NSLog(@"-----------------------------------------------------------");
    
    //第二种方式，不需要关闭objc_msgSend严格检查
    
    //1.无参无返回值
    ((void (*)(id, SEL))objc_msgSend)(self, NSSelectorFromString(@"run1"));
    
    //2.有参无返回值
    ((void (*)(id, SEL, id))objc_msgSend)(self, NSSelectorFromString(@"run2:"), @[ @"asdf", @12 ]);
    
    //3.无参有返回值
    ((id (*)(id, SEL))objc_msgSend)(self, NSSelectorFromString(@"run3"));
    
    //4.有参有返回值
    ((id (*)(id, SEL, id, id, id))objc_msgSend)(self, NSSelectorFromString(@"run4:age:height:"),  @"qwerty", @22, @60.8);
        
}

@end
```

Easy to use, doesn't it? 😊

