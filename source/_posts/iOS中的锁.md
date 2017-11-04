---
title: iOS中的锁
date: 2017-10-15 17:30:34
tags: NSLock, NSCondition, NSRecursiveLock, NSConditionLock, pthread_mutex, dispatch_semaphore
categories: iOS
---

## 简介
在`多线程编程`中，并发会使`一段代码`在`同一段时间`内线程之间互相`争抢资源`（`资源共享`）而产生`数据`的`不一致性`，为了`解决`这个问题，就`引入`了`锁`。锁的类型有多种，在iOS中，有如下：
- 1.OSSpinLock    自旋锁
- 2.dispatch_semaphore    GCD信号量实现加锁
- 3.pthread_mutex    互斥锁
- 4.NSLock  互斥锁
- 5.NSCondition    信号锁
- 6.pthread_mutex(recursive)  递归互斥锁
- 7.NSRecursiveLock  递归锁
- 8.NSConditionLock  条件锁
- 9.@synchronized  互斥锁

在看本篇文章前，请先了解[`GCD`](/2017/10/15/GCD的一般认知/)和[`NSOperation`](/2017/10/15/NSOperation的认知/), 如果你已熟知，请继续往下看。

我们先来看下iOS中全部的锁，以及它们的效率
![iOS 中全部的锁](/img/iOS/lock/iOS_lock_summary_benchmark.png)

这个简单的性能测试是在iPhone 6, iOS 9上跑的，[测试者在这篇文章](https://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/)
该结果显示的，横向柱状条最短的为性能最佳和最高；可知，OSSpinLock最佳，但是OSSpinLock被发现bug，Apple工程师透露了这个自旋锁有问题，暂时停用了，[查看这里](https://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/)
虽然OSSpinLock（自旋锁）有问题，但是我们还是看到了`pthread_mutex`和`dispatch_semaphore`性能排行仍是很高，而且苹果在新系统中也已经优化了
这两个锁的性能，所以我们在开发时也可以使用它们啦。

下面来一一介绍它们的使用

<br/>
## 1.dispatch_semaphore    GCD信号量实现加锁
GCD中提供了一种`信号机制`，也是为了`解决`资源`抢占问题`的，支持`信号通知`和`信号等待`。
- 1.每当`发送`一个`信号`时，则`信号量加1`
- 2.每当`发送`一个`等待信号`时，则`信号量减1`
- 3.如果`信号量为0`，则信号会处于`等待状态`，直到信号量`大于0`时就开始执行

```
- (void)example {

    //假设一共电影票3张票
    self.movieTickets = 3;
    
    //创建信号量
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    
    //添加任务1
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self buyTicketWithCounts:2 taskName:@"任务1" semaphore:semaphore];
    });
    
    //添加任务2
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self buyTicketWithCounts:2 taskName:@"任务2" semaphore:semaphore];
    });
}

- (void)buyTicketWithCounts:(int)counts taskName:(NSString *)taskName semaphore:(dispatch_semaphore_t)semaphore {
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    for (int i = 0; i < counts; i++) {
        if (self.movieTickets == 0) {
            NSLog(@"%@ 票已卖完! %@", taskName, [NSThread currentThread]);
            break;
        }
        
        NSLog(@"%@ 抢到%d票 剩余%d张票 %@", taskName, i + 1, --self.movieTickets, [NSThread currentThread]);
    }
    dispatch_semaphore_signal(semaphore);
}
```
输出结果如下
```
2017-10-16 test[23790:1042584] 任务1 抢到1票 剩余2张票 <NSThread: 0x604000465180>{number = 3, name = (null)}
2017-10-16 test[23790:1042584] 任务1 抢到2票 剩余1张票 <NSThread: 0x604000465180>{number = 3, name = (null)}
2017-10-16 test[23790:1042582] 任务2 抢到1票 剩余0张票 <NSThread: 0x604000464e00>{number = 4, name = (null)}
2017-10-16 test[23790:1042582] 任务2 票已卖完! <NSThread: 0x604000464e00>{number = 4, name = (null)}
```


<br/>
## 2.pthread_mutex    互斥锁
在POSIX（可移植操作系统）中，`pthread_mutex`是一套用于多线程同步的mutex锁，如同名一样，使用起来非常简单，性能比较高
``` 
     //初始化互斥锁
    __block pthread_mutex_t _mutex;
    pthread_mutex_init(&_mutex, NULL);
    
    //创建队列组
    dispatch_group_t group = dispatch_group_create();
    
    //创建并行队列
    dispatch_queue_t concurrentQueue = dispatch_queue_create("my.concurrent.queue", DISPATCH_QUEUE_CONCURRENT);
    
    //添加任务A到队列组
    dispatch_group_async(group, concurrentQueue, ^{
        pthread_mutex_lock(&_mutex);
        NSLog(@"NSBlockOperation A %@", [NSThread currentThread]);
        pthread_mutex_unlock(&_mutex);
    });
    
    //添加任务B到队列组
    dispatch_group_async(group, concurrentQueue, ^{
        pthread_mutex_lock(&_mutex);
        NSLog(@"NSBlockOperation B %@", [NSThread currentThread]);
        pthread_mutex_unlock(&_mutex);
    });
 
    //任务执行完，接收到通知
    dispatch_group_notify(group, concurrentQueue, ^{
        pthread_mutex_destroy(&_mutex);
        NSLog(@"pthread_mutex_t has been destroyed!");
    });
```
输出结果:
```
2017-10-16 test[22982:1011384] NSBlockOperation B <NSThread: 0x60000026a380>{number = 3, name = (null)}
2017-10-16 test[22982:1011382] NSBlockOperation A <NSThread: 0x604000465ac0>{number = 4, name = (null)}
2017-10-16 test[22982:1011382] pthread_mutex_t has been destroyed!
```


<br/>
## 3.NSLock  互斥锁
在Cocoa中`NSLock`是一种简单的互斥锁，继承自`NSLocking`协议，定义了`lock`和`unlock`方法，
而`NSLock`类还增加了`tryLock`和`lockBeforeDate:`方法。
- 1.`tryLock`方式试图获取一个锁，但是如果锁不可用的时候，它不会阻塞线程，相反它只会返回NO
- 2.`lockBeforeDate:`方法试图获取一个锁，但是如果锁没有在规定的时间内被获得，它会从阻塞状态变为非阻塞状态，返回NO
- 3.使用时，注意`lock`和`unlock`是成对出现的，也就说`lock`方法连续不能调用多次

我们这里来个简单的题：
假设一共有5张电影票，
现在有三个人去买票，每人要购买2张，
也就是三个人一共要买6张票，可是总电影票数只有5张，
所以最终他们有一人只能买到一张票
```
- (void)example {
    //创建锁的对象
    self.lock = [[NSLock alloc] init];
    
    //假设总共有5张电影票
    self.movieTickets = 5;
    
    //创建一个并行队列
    dispatch_queue_t myconcurrent = dispatch_queue_create("com.concurrent.queue.hello", DISPATCH_QUEUE_CONCURRENT);
    
    //A线程异步并行 买2张票
    dispatch_async(myconcurrent, ^{
        [self buyTicketWithCounts:2 thread:@"线程A"];
    });
    
    //B线程异步并行 买2张票
    dispatch_async(myconcurrent, ^{
        [self buyTicketWithCounts:2 thread:@"线程B"];
    });
    
    //C线程异步并行 买2张票
    dispatch_async(myconcurrent, ^{
        [self buyTicketWithCounts:2 thread:@"线程C"];
    });
}

- (void)buyTicketWithCounts:(int)counts thread:(NSString *)threadName {
    [self.lock lock];
    for (int i = 1; i <= counts; i++) {
        if (self.movieTickets == 0) {
            NSLog(@"票卖完了 %@", threadName);
            return;
        }
        NSLog(@"剩余票数：%d  %@ %@", self.movieTickets, threadName, [NSThread currentThread]);
        self.movieTickets--;
    }
    [self.lock unlock];
}
```
输出结果如下：
```
2017-10-16 test[20232:919739] 剩余票数：5  线程A <NSThread: 0x600000468240>{number = 3, name = (null)}
2017-10-16 test[20232:919739] 剩余票数：4  线程A <NSThread: 0x600000468240>{number = 3, name = (null)}
2017-10-16 test[20232:919738] 剩余票数：3  线程B <NSThread: 0x60000007fa40>{number = 4, name = (null)}
2017-10-16 test[20232:919738] 剩余票数：2  线程B <NSThread: 0x60000007fa40>{number = 4, name = (null)}
2017-10-16 test[20232:919745] 剩余票数：1  线程C <NSThread: 0x6040004674c0>{number = 5, name = (null)}
2017-10-16 test[20232:919745] 票卖完了 线程C
```
保证了总票数5张没有变，最终有一个人只能买到一张票


<br/>
## 4.NSCondition    信号锁
`NSCondition`也是派生自`NSLocking`, 所以它就有`lock`和`unlock`方法，但是`NSCondition`本身还有`wait`和`signal`方法，非常好用。
我们拿生产者消费者模式来举例吧
- 1.消费者获取锁，取产品，如果没有取到，则`wait`，这时会释放锁，知道有线程唤醒它去消费产品
- 2.生产者制造产品，首先也要取得锁，然后生产，再发`signal`，这样就可以唤醒正在`wait`的线程的消费者

```
- (void)ProducerConsumerPattern {
    self.products = [[NSMutableArray alloc] init];
    
    //创建信号量锁
    NSCondition *condition = [[NSCondition alloc] init];
    
    //创建一个并行队列
    NSOperationQueue *myQueue = [[NSOperationQueue alloc] init];
    
    //消费者
    NSBlockOperation *consumer = [NSBlockOperation blockOperationWithBlock:^{
        [condition lock];
        while (self.products.count == 0) {
            [condition wait]; //阻塞住，让线程等待，直到被通知到
        }
        
        NSLog(@"Consumed a product which named %@ %@", self.products.firstObject, [NSThread currentThread]);
        [condition unlock];
    }];
    
    //生产者
    NSBlockOperation *producer = [NSBlockOperation blockOperationWithBlock:^{
        [condition lock];
        
        NSString *productName = [NSString stringWithFormat:@"产品-%ld", random()];
        NSLog(@"Produced a product %@ %@ ", productName, [NSThread currentThread]);
        [self.products addObject:productName];
        
        [condition signal];
        [condition unlock];
    }];
    
    [myQueue addOperation:producer];
    [myQueue addOperation:consumer];
    
}
```
输出如下：
```
2017-10-16 test[24877:1088668] Produced a product 产品-1804289383 <NSThread: 0x600000269a00>{number = 3, name = (null)}
2017-10-16 test[24877:1088667] Consumed a product which named 产品-1804289383 <NSThread: 0x604000278700>{number = 4, name = (null)}
```


<br/>
## 5.pthread_mutex(recursive)  递归互斥锁
其实就是一个参数来断定`pthread_mutex_t`是否是递归锁，
我们先来看下死锁的例子
```
- (void)pthread_recursive_lock {
    __block pthread_mutex_t _mutext;
    pthread_mutex_init(&_mutext, NULL);
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        static void (^MyBlock)(int);
        MyBlock = ^(int value){
            pthread_mutex_lock(&_mutext); //第二次运行到这里会阻塞住，产生死锁，因为之前被锁住的资源还未解锁，所以就造成它们俩互相等待
            if (value > 0) {
                NSLog(@"value = %d %@", value, [NSThread currentThread]);
                MyBlock(value - 1);
            }
        };
        MyBlock(5);
        pthread_mutex_unlock(&_mutext);
    });
}
```
解决这个死锁的重点就是给pthread_mutex_t设置属性为递归锁，代码如下
```
- (void)pthread_recursive_lock {

    //创建互斥锁的属性对象，并设置递归锁
    pthread_mutexattr_t _mutexattr;
    pthread_mutexattr_init(&_mutexattr);
    pthread_mutexattr_settype(&_mutexattr, PTHREAD_MUTEX_RECURSIVE);
    
    //创建互斥锁对象
    __block pthread_mutex_t _mutext;
    pthread_mutex_init(&_mutext, &_mutexattr);
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        static void (^MyBlock)(int);
        MyBlock = ^(int value){
            pthread_mutex_lock(&_mutext); //第二次运行到这里会产生死锁，因为之前被锁住的资源还未解锁，所以就造成它们俩互相等待
            if (value > 0) {
                NSLog(@"value = %d %@", value, [NSThread currentThread]);
                MyBlock(value - 1);
            }
        };
        MyBlock(5);
        pthread_mutex_unlock(&_mutext);
        pthread_mutex_destroy(&_mutext);
    });
}
```
输出结果如下：
```
2017-10-16 test[25369:1103912] value = 5 <NSThread: 0x600000464a00>{number = 3, name = (null)}
2017-10-16 test[25369:1103912] value = 4 <NSThread: 0x600000464a00>{number = 3, name = (null)}
2017-10-16 test[25369:1103912] value = 3 <NSThread: 0x600000464a00>{number = 3, name = (null)}
2017-10-16 test[25369:1103912] value = 2 <NSThread: 0x600000464a00>{number = 3, name = (null)}
2017-10-16 test[25369:1103912] value = 1 <NSThread: 0x600000464a00>{number = 3, name = (null)}
```


<br/>
## 6.NSRecursiveLock  递归锁
`NSRecursiveLock`是一个递归锁，它的`lock`方法可以被同一个线程多次请求，而且不会引起死锁；
主要用在循环或者递归操作中，多次`lock`，只需要一次`unlock`，因为递归锁内部会有一个跟踪被`lock`的数次的功能，
不管被`lock`多少次，最后`unlock`也会把所有的持有资源给解锁，来看一个经典的死锁案例，如下
```
    NSLock *lock_i = [[NSLock alloc] init];
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        static void (^MyBlock)(int);
        MyBlock = ^(int value) {
            [lock_i lock]; //加锁代码在递归执行第二次时阻塞了，也就是死锁了
            if (value > 0) {
                NSLog(@"value = %d %@", value, [NSThread currentThread]);
                sleep(2);
                MyBlock(value - 1);
            }
            [lock_i unlock];
        };
        MyBlock(5);
    });
```
看看这个代码，由于在递归运行过程中，`[lock_i lock];`会被多次调用，而`NSLock`每次`lock`对象时，必须是`unlock`状态，
所以它就会一直等着上一个`lock`的对象资源被`unlock`掉，但是上一个并没有执行`unlock`，所以就造成了他们之间互相等待，而形成死锁。
为了解决这个问题，我们就需要使用递归锁`NSRecursiveLock`，因为递归锁可以多次`lock`，最后一次`unlock`就能解锁所有已经被`lock`的对象
```
    NSRecursiveLock *lock = [[NSRecursiveLock alloc] init];
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        static void (^MyBlock)(int);
        MyBlock = ^(int value) {
            [lock lock]; //这行代码加锁执行了多次
            if (value > 0) {
                NSLog(@"value = %d %@", value, [NSThread currentThread]);
                sleep(2);
                MyBlock(value - 1);
            }
            [lock unlock];//解锁只执行了一次
        };
        MyBlock(5);
    });
```
输出结果为：
```
2017-10-16 test[21404:957416] value = 5 <NSThread: 0x604000073280>{number = 3, name = (null)}
2017-10-16 test[21404:957416] value = 4 <NSThread: 0x604000073280>{number = 3, name = (null)}
2017-10-16 test[21404:957416] value = 3 <NSThread: 0x604000073280>{number = 3, name = (null)}
2017-10-16 test[21404:957416] value = 2 <NSThread: 0x604000073280>{number = 3, name = (null)}
2017-10-16 test[21404:957416] value = 1 <NSThread: 0x604000073280>{number = 3, name = (null)}
```


<br/>
## 7.NSConditionLock  条件锁
`NSConditionLock`定义了一组可以指定`int类型`条件的互斥锁
```
    NSConditionLock *conditionLock = [[NSConditionLock alloc] initWithCondition:0];
    
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        for (int i = 0; i <= 3; i++) {
            [conditionLock lock];
            NSLog(@"A %d %@", i, [NSThread currentThread]);
            sleep(1);
            [conditionLock unlockWithCondition:i];
        }
    });
    
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
        [conditionLock lock];
        NSLog(@"B %@",[NSThread currentThread]);
        [conditionLock unlock];
    });
```


<br/>
## 8.@synchronized  互斥锁
我们这里来个简单的题：
假设一共有5张电影票，
现在有三个人去买票，每人要购买2张，
也就是三个人一共要买6张票，可是总电影票数只有5张，
所以最终他们有一人只能买到一张票

`@synchronized`关键字加锁，是一种互斥锁，性能较差不推荐使用；看代码示例：
```
- (void)example {
    //假设总共有5张电影票
    self.movieTickets = 5;
    
    //创建一个并行队列
    dispatch_queue_t myconcurrent = dispatch_queue_create("com.concurrent.queue.hello", DISPATCH_QUEUE_CONCURRENT);
    
    //A线程异步并行 买2张票
    dispatch_async(myconcurrent, ^{
        [self buyTicketWithCounts:2 thread:@"线程A"];
    });
    
    //B线程异步并行 买2张票
    dispatch_async(myconcurrent, ^{
        [self buyTicketWithCounts:2 thread:@"线程B"];
    });
    
    //C线程异步并行 买2张票
    dispatch_async(myconcurrent, ^{
        [self buyTicketWithCounts:2 thread:@"线程C"];
    });
}

- (void)buyTicketWithCounts:(int)counts thread:(NSString *)threadName {
    @synchronized(self) {
        for (int i = 1; i <= counts; i++) {
            if (self.movieTickets == 0) {
                NSLog(@"票卖完了 %@", threadName);
                return;
            }
            NSLog(@"剩余票数：%d  %@ %@", self.movieTickets, threadName, [NSThread currentThread]);
            self.movieTickets--;
        }
    }
}
```
猜猜输出结果会是什么？
```
2017-10-16 test[19868:910931] 剩余票数：5  线程A <NSThread: 0x600000270400>{number = 3, name = (null)}
2017-10-16 test[19868:910931] 剩余票数：4  线程A <NSThread: 0x600000270400>{number = 3, name = (null)}
2017-10-16 test[19868:910928] 剩余票数：3  线程B <NSThread: 0x600000270640>{number = 4, name = (null)}
2017-10-16 test[19868:910928] 剩余票数：2  线程B <NSThread: 0x600000270640>{number = 4, name = (null)}
2017-10-16 test[19868:910930] 剩余票数：1  线程C <NSThread: 0x6000002705c0>{number = 5, name = (null)}
2017-10-16 test[19868:910930] 票卖完了 线程C
```
这里例子说明，总票数5张没有变，因为使用了`@synchronized`互斥锁；假设此时，我们不用`@synchronized`，会输出什么结果了？
```
2017-10-16 test[19984:914005] 剩余票数：5  线程A <NSThread: 0x604000067c40>{number = 4, name = (null)}
2017-10-16 test[19984:914004] 剩余票数：5  线程C <NSThread: 0x600000276180>{number = 3, name = (null)}
2017-10-16 test[19984:914007] 剩余票数：5  线程B <NSThread: 0x60400026c880>{number = 5, name = (null)}
2017-10-16 test[19984:914005] 剩余票数：4  线程A <NSThread: 0x604000067c40>{number = 4, name = (null)}
2017-10-16 test[19984:914004] 剩余票数：3  线程C <NSThread: 0x600000276180>{number = 3, name = (null)}
2017-10-16 test[19984:914007] 剩余票数：2  线程B <NSThread: 0x60400026c880>{number = 5, name = (null)}
```
看到没，卖出了6张票😰




<br/>
<br/>
[GCD的一般认知，打开](/2017/10/15/GCD的一般认知/)
[NSOperation的认知，打开](/2017/10/15/NSOperation的认知/)
[NSThread的认知](/2017/10/15/NSThread的认知/)
[iOS的Runloop认知，打开](/2017/10/15/iOS的Runloop认知/)



