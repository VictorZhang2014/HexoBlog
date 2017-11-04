---
title: GCD的一般认知
date: 2017-10-15 17:04:21
tags: iOS, GCD, 队列, 死锁
categories: iOS
---

## GCD （Grand Central Dispatch）
GCD两个核心概念：`任务`和`队列

#### **任务**
`任务`就是`执行操作`的意思，也就是`block那段代码`。执行操作有两种：`同步执行`和`异步执行`。
同步执行（sync）：阻塞主线程并执行任务，不会开启新线程任务
异步执行（async）：不会阻塞主线程，会开启新线程执行任务，在后台执行

#### **队列**
这里的队列就是`任务队列`，即用来存放任务的队列。队列是一种特殊的线性表，采用先进先出（FIFO）的原则，
每次`新任务`都会被`插入`到`队列尾部`，而`执行队列`中的`任务`时，会从队列`头部`开始`读取`并`执行`。
GCD中有两种队列：`串行队列`和`并行队列`
1.并行队列（DISPATCH_QUEUE_CONCURRENT）：可以多个任务同时进行，也就会开启多个线程执行任务。交替执行。
2.串行队列（DISPATCH_QUEUE_SERIAL）：任务一个接着一个执行，也就是一个任务执行完后，下一个任务就开始。一个接着一个执行。

#### **队列的创建**
```
// 串行队列
dispatch_queue_t queue= dispatch_queue_create("my_queue_serial", DISPATCH_QUEUE_SERIAL);

// 并行队列
dispatch_queue_t queue= dispatch_queue_create("my_queue_concurrent", DISPATCH_QUEUE_CONCURRENT);
```

#### **GCD默认提供了`全局队列`和`主队列`**
1.全局队列 `dispatch_get_global_queue` ，全局队列就是并行队列，供整个应用使用；
  需要两个参数，第一个是队列优先级（`DISPATCH_QUEUE_PRIORITY_DEFAULT`），第二个0即可(官方文档说：For future use)
2.主队列 `dispatch_get_main_queue` ，主队列就是串行队列，在应用启动时，就创建好了，所以我们要用的时候就直接拿来用而不需要创建

#### **任务和队列的组合**
1.并行队列 + 同步执行 
2.并行队列 + 异步执行
3.串行队列 + 同步执行
4.串行队列 + 异步执行

还有两个特殊组合
1.主队列 + 同步执行（`会死锁并崩溃`）
2.主队列 + 异步执行

|                |             并行队列                  |            串行队列                 |                        主队列                        |
| -------------- |:------------------------------------:|:----------------------------------:|:--------------------------------------------------:|
| 同步（顺序执行）  | 阻塞主线程，没有开启新线程，串行执行任务    | 阻塞主线程，没有开启新线程，串行执行任务 |  阻塞主线程，没有开启新线程，串行执行任务（会死锁导致崩溃）  |
| 异步（并发执行）  | 不阻塞主线程，有开启新线程，并行执行任务    | 不阻塞主线程，有开启新线程，并行执行任务 | 不阻塞主线程，没有开启新线程，串行执行任务                 |

#### **看一看几种死锁原因**
```
// 有死锁案例1
- (void)threadDeadLockCase1 {
    //这是个死锁案例，两个串行同步操作的队列嵌套会导致第二个串行同步队列运行不了（也就会产生死锁并崩溃）
    //原因：因为第一个串行同步队列打印完 `NSLog(@"2");` 后，就在等待第二个串行同步操作队列执行完
    //     但是第二个串行同步队列需要等待第一个串行同步操作队列执行完，方可继续，
    //     最终，两个串行同步队列互相等待，导致死锁
    dispatch_queue_t queue = dispatch_queue_create("my_queue_serial", DISPATCH_QUEUE_SERIAL);
    NSLog(@"1");
    dispatch_sync(queue, ^{
        NSLog(@"2");
        dispatch_sync(queue, ^{  //死锁，会崩溃掉
            NSLog(@"3");
        });
        NSLog(@"4");
    });
    NSLog(@"5");

    //最终会打印出1，2，然后崩溃
}

// 有死锁案例2
- (void)threadDeadLockCase2 {
    //原因：因为同步串行队列在异步执行操作里
    dispatch_queue_t queue = dispatch_queue_create("my_queue_serial", DISPATCH_QUEUE_SERIAL);
    NSLog(@"1 %@", [NSThread currentThread]);
    dispatch_async(queue, ^{
        NSLog(@"2 %@", [NSThread currentThread]);
        dispatch_sync(queue, ^{  //死锁并崩溃掉
            NSLog(@"3 %@", [NSThread currentThread]);
        });
        NSLog(@"4");
    });
    NSLog(@"5");
    
    //会打印1，3，2 然后死锁并崩溃
}

// 有死锁案例3
- (void)threadDeadLockCase3 {
    //因为主线程其实就是在串行队列里，这个例子和上面那个方法（threadDeadLockCase1()）串行队列嵌套两个同步线程的原理是一样的
    //这个例子就是两个主线程串行队列嵌套
    NSLog(@"1");
    dispatch_sync(dispatch_get_main_queue(), ^ { //会崩溃，这就相当于在主线程里嵌套了一个同步主线程队列，也就会产生循环死锁
        NSLog(@"2");
    });
}
```

#### **GCD线程之间的通讯**
在iOS开发过程中，我们一般在主线程里边进行UI刷新，例如：点击、滚动、拖拽等事件。我们通常把一些耗时的操作放在其他线程，
比如：图片下载、文件上传等耗时操作。而当我们有时候在其他线程完成了耗时操作时，需要回到主线程，那么就用到了线程之间的通讯。
```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    //做某些下载操作

    // 回到主线程
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"更新UI"]);
    });
});
```

#### **GCD的其他方法**
#### *GCD的栅栏方法 `dispatch_barrier_async`*
我们有时候需要异步执行两组操作，而且第一组操作执行完之后，才能开始执行第二组操作。这样我们就需要一个相当于`栅栏`一样的一个方法将两组异步执行的
操作分割开来，当然这里的操作组里可以包含一个或多个任务。这就需要用到`dispatch_barrier_async`方法在两个操作组间形成栅栏。
```
- (void)barrierAsync {
    dispatch_queue_t myconcurrent = dispatch_queue_create("my_queue_concurrent", DISPATCH_QUEUE_CONCURRENT);
    
    //第一组 并行队列异步操作
    dispatch_async(myconcurrent, ^{
        NSLog(@"1 %@", [NSThread currentThread]);
    });
    dispatch_async(myconcurrent, ^{
        NSLog(@"2 %@", [NSThread currentThread]);
    });
    
    //只有第一组执行完后，第二组才会开始执行
    dispatch_barrier_sync(myconcurrent, ^{
        NSLog(@"barrier_sync");
    });
    
    //第二组 并行队列异步操作
    dispatch_async(myconcurrent, ^{
        NSLog(@"3 %@", [NSThread currentThread]);
    });
    dispatch_async(myconcurrent, ^{
        NSLog(@"4 %@", [NSThread currentThread]);
    });
}
```
输出结果是：
```
[7017:431987] 2 <NSThread: 0x6000002756c0>{number = 4, name = (null)}
[7017:431986] 1 <NSThread: 0x604000461700>{number = 3, name = (null)}
[7017:431702] barrier_sync
[7017:431987] 4 <NSThread: 0x6000002756c0>{number = 4, name = (null)}
[7017:431988] 3 <NSThread: 0x604000461440>{number = 5, name = (null)}
```

<br/>
#### *GCD的延时执行方法 `dispatch_after`*
当我们需要延迟执行一段代码时，就需要用到GCD的 `dispatch_after` 方法。
```
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"三秒后，异步执行这里的代码");
    });
```

<br/>
#### *GCD的只执行一次方法 `dispatch_once`*
常用于创建单例时使用，也就是在整个应用程序运行过程中`dispatch_once`的block任务只会被执行一次
```
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        NSLog(@"这个block任务只会被执行一次");
    });
```

<br/>
#### *GCD的快速迭代方法 `dispatch_apply`*
通常我们会使用 `for` 循环遍历，但是GCD给我们提供了一个快速迭代的方法 `dispatch_apply` 使我们可以同时遍历。
比如：说遍历0~5 这6个数字，for循环就是每次取出一个元素进行遍历，但是 `dispatch_apply`却是同时遍历的
```
    dispatch_queue_t global_queue = dispatch_get_global_queue(0, 0);
    dispatch_apply(6, global_queue, ^(size_t index) {
        NSLog(@"%zd %@", index, [NSThread currentThread]);
    });
```
看输出结果的时间，我们可以得知，6个数字是同时迭代完的
```
2017-10-15 16:22:31.807072+0800 test[7302:444592] 0 <NSThread: 0x6000000712c0>{number = 1, name = main}
2017-10-15 16:22:31.807073+0800 test[7302:444696] 1 <NSThread: 0x60400026f780>{number = 3, name = (null)}
2017-10-15 16:22:31.807072+0800 test[7302:444698] 3 <NSThread: 0x60000027e580>{number = 5, name = (null)}
2017-10-15 16:22:31.807109+0800 test[7302:444697] 2 <NSThread: 0x60400026f600>{number = 4, name = (null)}
2017-10-15 16:22:31.807266+0800 test[7302:444592] 4 <NSThread: 0x6000000712c0>{number = 1, name = main}
2017-10-15 16:22:31.807274+0800 test[7302:444696] 5 <NSThread: 0x60400026f780>{number = 3, name = (null)}
```

<br/>
#### *GCD的队列组 `dispatch_group_t`*
有时候我们会有这样的需求：分别异步执行几个耗时的操作，然后当这几个耗时的操作都执行完毕后，再回到主线程执行操作，这时我们就需要用到队列组了。
```
    //全局队列
    dispatch_queue_t global_queue = dispatch_get_global_queue(0, 0);
    
    //创建一个队列组
    dispatch_group_t group = dispatch_group_create();
    
    //将block操作加入到任务组
    dispatch_group_enter(group);
    dispatch_group_async(group, global_queue, ^{
        NSLog(@"执行第一个耗时的任务操作 %@", [NSThread currentThread]);
        
        //该任务执行完操作后，就马上从任务组中移除
        dispatch_group_leave(group);
    });
    
    dispatch_group_enter(group);
    dispatch_group_async(group, global_queue, ^{
        NSLog(@"执行第二个耗时的任务操作 %@", [NSThread currentThread]);
        dispatch_group_leave(group);
    });
    
    dispatch_group_enter(group);
    dispatch_group_async(group, global_queue, ^{
        NSLog(@"执行第三个耗时的任务操作 %@", [NSThread currentThread]);
        dispatch_group_leave(group);
    });
    
    //上面的任务都执行完后，会有以下两种方式来处理结果
    //第一种 会阻塞主线程，等待上面的任务执行完，再继续向下执行
    //dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    
    //第二种 不会阻塞主线程，等待上面的任务执行完，该block就会执行 (推荐)
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"回到主线程 %@", [NSThread currentThread]);
    });
```
输出的结尾如下，无论如何当所有的任务执行完后，dispatch_group_notify里的block就是最后执行的，因为是并行队列，所以它们的顺序不会一致的
```
2017-10-15 17:00:51.710362+0800 test[7983:475352] 执行第二个耗时的任务操作 <NSThread: 0x60000027fb40>{number = 3, name = (null)}
2017-10-15 17:00:51.710362+0800 test[7983:475358] 执行第三个耗时的任务操作 <NSThread: 0x60000027fbc0>{number = 4, name = (null)}
2017-10-15 17:00:51.710418+0800 test[7983:475354] 执行第一个耗时的任务操作 <NSThread: 0x60000027fd00>{number = 5, name = (null)}
2017-10-15 17:00:51.719487+0800 test[7983:475038] 回到主线程 <NSThread: 0x60400007c680>{number = 1, name = main}
```


<br/>
[NSOperation的认知，打开](/2017/10/15/NSOperation的认知/)
[iOS中的锁，打开](/2017/10/15/iOS中的锁/)
[NSThread的认知，打开](/2017/10/15/NSThread的认知/)
[iOS的Runloop认知，打开](/2017/10/15/iOS的Runloop认知/)


