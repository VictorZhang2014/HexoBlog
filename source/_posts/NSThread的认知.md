---
title: NSThread的认知
date: 2017-10-15 18:36:14
tags: iOS, NSThread
---

## 简介
`NSThread`多线程编程，超级简单，NSthread是基于pthread_t封装的，所以基本上在使用方面pthread_t和NSThread差不多

线程的生命周期，五种状态
- 1.新建（new Thread）,就是实例化了一个线程对象
    在iOS中，`self.alwasyThread = [[NSThread alloc] initWithTarget:self selector:@selector(alwaysRun) object:nil];`
- 2.就绪（runnable），就是线程在就绪队列中等待CPU分配时间片，一般是`start`方法
    在iOS中，`[self.alwasyThread start];`
- 3.运行（running），就是线程已经获得CPU资源并且马上执行任务，一般是`run`方法
    在iOS中，`start`方法就表示进入就绪状态，并且获得CPU资源后进入运行状态
- 4.死亡（dead），就是线程执行完任务，或者被其他线程杀死，这时就不能再进入就绪状态，重新运行。调用`stop`方法终止线程
    在iOS中，`[NSThread exit];`
- 5.阻塞（blocked），就是某种原因导致正在运行的线程暂停自己，让出CPU，那么自己就进入了阻塞状态（suspend），阻塞状态可以调用`resume`恢复
    在iOS中，`sleep(3);`，`[NSThread sleepForTimeInterval:3.0f];`，`[NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:3.0]];`
   

<br/>
我们分三步说下吧

## 1.创建子线程
第一种方式
```
- (void)nsthread_test {
    NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];
    [thread start];
}

- (void)run {
    NSLog(@"NSThread子线程 %@", [NSThread currentThread]);
}
```
输出为：
```
2017-10-16 test[25785:1117888] NSThread子线程 <NSThread: 0x604000270200>{number = 3, name = (null)}
```

第二种方式，仅限iOS 10及以上版本可用
```
    NSThread *thread = [[NSThread alloc] initWithBlock:^{
        NSLog(@"NSThread子线程 %@", [NSThread currentThread]);
    }];
    [thread start];
```

<br/>
## 2.分离线程
```
- (void)nsthread_test {
    [NSThread detachNewThreadSelector:@selector(run:) toTarget:self withObject:@[ @"这是", @"参数"]];
}

- (void)run:(id)parameters {
    NSLog(@"NSThread子线程 parameter=%@ %@", parameters, [NSThread currentThread]);
}
```
输出为：
```
2017-10-16 test[25940:1121175] NSThread子线程 parameter=("这是", "参数") <NSThread: 0x604000460240>{number = 3, name = (null)}
```


<br/>
## 3.后台线程
```
- (void)nsthread_test {
    [self performSelectorInBackground:@selector(run:) withObject:@[ @"这是", @"参数"]];
}

- (void)run:(id)parameters {
    NSLog(@"NSThread后台线程 parameter=%@ %@", parameters, [NSThread currentThread]);
}
```
输出为：
```
2017-10-16 test[25940:1127130] NSThread后台线程 parameter=("这是", "参数") <NSThread: 0x6000002718c0>{number = 3, name = (null)}
```


还有几个方法都是通过`self`调用的
```
//在主线程上执行
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;

//在指定线程上执行
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
```


<br/>
<br/>
[GCD的一般认知](/2017/10/15/GCD的一般认知/)
[NSOperation的认知](/2017/10/15/NSOperation的认知/)
[iOS中的锁](/2017/10/15/iOS中的锁/)
[iOS的Runloop认知](/2017/10/15/iOS的Runloop认知/)

