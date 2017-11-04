---
title: iOS的Runloop认知
date: 2017-10-15 19:35:32
tags: iOS, Runloop
categories: iOS
---

## 概念
`RunLoop`是iOS和OS X开发中非常基础的知识，通过RunLoop可以实现自动释放池，延迟回调，触摸事件，屏幕刷新等功能。

一般来讲，一个线程一次只能执行一个任务，执行完成后线程就会退出。如果我们需要一个机制，让线程能随时处理事件但并不退出，通常的代码如下：
```
function loop() {
    initialize();
    do {
        var message = get_next_message();
        process_message(message);
    } while (message != quit);
}
```
这种模型通常被称作 `Event Loop`。 `Event Loop` 在很多系统和框架都有实现，比如Node.js的事件处理，比如Windows程序消息循环，再比如iOS/OS X里的RunLoop.
实现这种模型的关键点在于：如何管理事件/消息，如何让线程在没有处理消息时休眠以避免资源占用、在有消息来到时立刻被唤醒。

所以 `RunLoop` 实际上就是一个对象，这个对象管理了其需要处理的事件和消息，并提供了一个入口函数来执行上面的 `Event Loop` 的逻辑。线程执行了这个函数后，就会一直处于这个函数内部 “接收消息 -> 等待 -> 处理” 的循环中，知道这个循环结束（比如传入`quit`的消息），函数返回。

在iOS/OS X系统中，提供了两个这样的对象：NSRunLoop和CFRunLoopRef。
`CFRunLoopRef`是在CoreFoundation框内的，提供了`纯C函数`的API，[代码是开源的](http://opensource.apple.com/tarballs/CF/)，所有这些API都是线程安全的。
`NSRunLoop`是基于`CFRunLoopRef`的封装，提供了`面向对象`的API，但是这些API不是线程安全的。

`Swift`开源后，苹果又维护了一个跨平台的`CoreFoundation`版本：https://github.com/apple/swift-corelibs-foundation/  这个版本的源码可能和现有的iOS系统中的实现略有不同，但是更容易编译，因为它已经适配了 `Linux/Windows`

<br/>

## RunLoop对外的接口
在`CoreFoundation`里面关于RunLoop有5个类
- CFRunLoopRef
- CFRunLoopModeRef
- CFRunLoopSourceRef
- CFRunLoopTimerRef
- CFRunLoopObserverRef

其中 CFRunLoopModeRef 类并没有对外暴露，只是通过 CFRunLoopRef 的接口进行了封装。他们的关系如下:
<img src="/img/iOS/RunLoop/RunLoop_0.png" alt="" style="width:350px;height:280px;" />

* 一个RunLoop包含若干个Mode
* `每个Mode`包含若干个`Source/Timer/Observer`
* `每次`调用`RunLoop的主函数`时，只能`指定`其中一个`Mode`，这个`Mode`被`称作CurrentMode`
* 如果需要切换Mode，只能先`退出Loop`，再`重新指定`一个`Mode`进入
* 这样做的`目的`是为了分割开不同组的`Source/Timer/Observer`

** CFRunLoopSourceRef **是事件产生的地方。Source有两个版本：`Source0`和`Source1`。
* `Source0` 包含了一个回调（函数指针），不会主动出发事件。使用时，需要先调用`CFRunLoopSourceSignal(source)` 将这个Source标记为待处理，然后手动调用`CFRunLoopWakeUp(runloop)`来唤醒`RunLoop`，让其处理这个事件。
* `Source1` 包含了一个`mach_port`和一个回调（函数指针），被用于通过内核和其他线程互相发送消息。这种Source能主动唤醒RunLoop的线程。

** CFRunLoopTimerRef ** 是基于时间的触发器，它和`NSTimer`是toll-free bridged（也就是互相可替换）的，可以混用；它包含了一个`时间长度`和`一个回调`；当其被加入到RunLoop时，RunLoop会注册对应的时间点，当达到时间点时，RunLoop会被环形以执行那个回调

** CFRunLoopObserverRef ** 是观察者，每一个Observer都有一个回调（函数指针），当RunLoop状态发生变化时，观察者就能通过回调接受到这个变化，观测的时间点有：
```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```

<br/>

## RunLoop的Mode
CFRunLoopMode结构如下
```
struct __CFRunLoopMode {
    CFStringRef _name;            // Mode Name, 例如 @"kCFRunLoopDefaultMode"
    CFMutableSetRef _sources0;    // Set
    CFMutableSetRef _sources1;    // Set
    CFMutableArrayRef _observers; // Array
    CFMutableArrayRef _timers;    // Array
    ...
};
```
CFRunLoop结构如下
```
struct __CFRunLoop {
    CFMutableSetRef _commonModes;     // Set
    CFMutableSetRef _commonModeItems; // Set<Source/Observer/Timer>
    CFRunLoopModeRef _currentMode;    // Current Runloop Mode
    CFMutableSetRef _modes;           // Set
    ...
};
```

苹果公开的Mode有两个，这两个Mode都是被标记为`common`属性，如下：
* kCFRunLoopDefaultMode(UIDefaultRunLoopMode)    
* UITrackingRunLoopmode                 

应用场景举例：
主线程的RunLoop的`UIDefaultRunLoopMode`是App平时所处的状态，`UITrackingRunLoopmode`是追踪`ScrollView`滑动时的状态。当你创建一个`Timer`并加入到`DefaultMode`时，Timer会得到重复回调，但是此时滑动一个TableView时，RunLoop会将Mode切换为TrackingRunLoopMode，这时Timer就不会被回调，并且也不会影响滑动操作。
可是有时你需要一个`Timer`，在两个mode中都能得到回调，办法有两种;
- 1.将这个`Timer`分别`加`入`到两个Mode`中去
- 2.将`Timer`加入到顶层的RunLoop的`commonMode`


<br/>

## RunLoop的内部逻辑
根据苹果文档里的说明，RunLoop内部的大概逻辑如下：
![RunLoop 内部逻辑](/img/iOS/RunLoop/RunLoop_1.png) 

具体看[这里](https://blog.ibireme.com/2015/05/18/runloop/)

<br/>

## 苹果用RunLoop实现的功能
首先我们来了解下App启动后的RunLoop的状态，分别向系统注册了5个mode：
- 1.kCFRunLoopDefaultMode，App的默认mode，通常主线程在这个mode下运行的
- 2.UITrackingRunLoopMode，界面跟踪mode，用于UIScrollView追踪触摸滑动时保证界面不受其他mode影响
- 3.UIInitializationRunLoopMode，在App刚启动时第一个进入的Mode，启动完后便不再使用
- 4.GSEventReceiveRunLoopMode，接受系统事件的内部mode，通常用不到
- 5.kCFRunLoopCommonModes，这是一个占位mode，没有实际作用

** 定时器 **
`NSTimer`其实就是`CFRunLoopTimerRef`，他们之间是toll-free bridged（互相替换）。一个`NSTimer`注册到RunLoop后，Runloop会为其重新的时间点注册好事件。
例如：10:00, 10:10, 10:20 这个几个时间点。RunLoop为了节省资源，并不会在非常准确的时间点回调这个Timer。Timer有个属性叫做Tolerance(宽容度)，表示当时间点到达后，容许有多少的误差。如果某一个时间点错过了，例如执行了一个很长时间的任务，则那个时间点的回调会被跳过去，不会延后执行

`CADisplayLink`是一个和屏幕刷新率一致的定时器（但实际实现原理更为复杂，和`NSTimer`并不一样）。如果在两次屏幕刷新之间执行了一个任务，那其中就会有一帧会被跳过去（和NSTimer一样），这就造成了界面卡顿的感觉。尤其是在快速滑动tableView时，即时有一帧的卡顿也会让用户有所察觉。Facebook开源了[`AsyncDisplayLink`](https://github.com/facebookarchive/AsyncDisplayKit)（现在改名了叫做[`Texture`](https://github.com/texturegroup/texture/)）就是为了解决界面卡顿的问题，其内部也用到了RunLoop。

** PerformSelector **
当调用`NSObject`的`performSelector:afterDelay:`后，实际上是在其内部创建了一个Timer并且加入到当前的线程的RunLoop中，所以如果当前线程中没有RunLoop，则这个方法会失效

当调用`performSelector:onThread:`时，实际上也会创建一个Timer加到对应的线程中去，同样的，如果对应的线程中没有RunLoop，则该方法也会失效

以上的内容摘自：https://blog.ibireme.com/2015/05/18/runloop/

<br/>
<br/>
## 看例子
第一个例子，让一个线程常驻
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSLog(@"1.创建线程");
    self.alwasyThread = [[NSThread alloc] initWithTarget:self selector:@selector(alwaysRun) object:nil];
    NSLog(@"2.启动线程，包括：1）.线程进入就绪状态；2）.线程获得CPU资源后运行状态");
    [self.alwasyThread start];
}

- (void)alwaysRun {
    NSLog(@"该线程一直在活跃 %@", [NSThread currentThread]);
    
    self.runloop = [NSRunLoop currentRunLoop];
    [self.runloop addPort:[NSPort port] forMode:NSDefaultRunLoopMode];
    [self.runloop run];
    
    NSLog(@"不会执行到这里");
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    [self performSelector:@selector(subthreadRun) onThread:self.alwasyThread withObject:nil waitUntilDone:NO];
}

- (void)subthreadRun {
    NSLog(@"你点击了屏幕 %@", [NSThread currentThread]);
    
    NSTimer *timer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(timerRun) userInfo:nil repeats:YES];
    [self.runloop addTimer:timer forMode:NSDefaultRunLoopMode];
}

- (void)timerRun {
    static int i = 0;
    NSLog(@"%d", i++);
    
    if (i == 5) {
        NSLog(@"3.线程进入阻塞状态，阻塞3秒钟");
        //    [NSThread sleepForTimeInterval:3.0f];
        [NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:3.0]];
        sleep(3);
        
        NSLog(@"4.退出线程，退出线程后，该方法下面的代码不在执行");
        [NSThread exit];
        
        NSLog(@"该线程挂了");
    }
}
```
输出了
```
2017-10-17 13:29:32.900140+0800 test[30877:1429252] 1.创建线程
2017-10-17 13:29:32.900374+0800 test[30877:1429252] 2.启动线程，包括：1）.线程进入就绪状态；2）.线程获得CPU资源后运行状态
2017-10-17 13:29:32.901087+0800 test[30877:1429358] 该线程一直在活跃 <NSThread: 0x604000461580>{number = 3, name = (null)}
2017-10-17 13:29:35.233913+0800 test[30877:1429358] 你点击了屏幕 <NSThread: 0x604000461580>{number = 3, name = (null)}
2017-10-17 13:29:36.236729+0800 test[30877:1429358] 0
2017-10-17 13:29:37.235340+0800 test[30877:1429358] 1
2017-10-17 13:29:38.237163+0800 test[30877:1429358] 2
2017-10-17 13:29:39.235978+0800 test[30877:1429358] 3
2017-10-17 13:29:40.240552+0800 test[30877:1429358] 4
2017-10-17 13:29:40.240877+0800 test[30877:1429358] 3.线程进入阻塞状态，阻塞3秒钟
2017-10-17 13:29:43.243757+0800 test[30877:1429358] 4.退出线程，退出线程后，该方法下面的代码不在执行
```
我们可以看到，一个`线程的生命周期`，从`开始`到`结束`，如果我们不点击屏幕的话，那么这个`线程`就是`一直常驻`的，当`点击`完屏幕后，`阻塞`三秒钟，就`退出线程`了，线程退出`runloop`也就`挂`了

<br/>
下面在来一个例子，监听runloop的状态
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSLog(@"%@ 1.创建线程", [NSThread currentThread]);
    self.alwasyThread = [[NSThread alloc] initWithTarget:self selector:@selector(alwaysRun) object:nil];
    NSLog(@"%@ 2.启动线程，包括：1）.线程进入就绪状态；2）.线程获得CPU资源后运行状态", [NSThread currentThread]);
    [self.alwasyThread start];
}

- (void)alwaysRun {
    NSLog(@"%@ 该线程一直在活跃", [NSThread currentThread]);
    
    CFRunLoopObserverRef runLoopObserver = CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(), kCFRunLoopAllActivities, true, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
        switch (activity) {
            case kCFRunLoopEntry:
                NSLog(@"%@ 即将进入 runloop", [NSThread currentThread]);
                break;
            case kCFRunLoopBeforeTimers:
                NSLog(@"%@ 即将处理 Timer", [NSThread currentThread]);
                break;
            case kCFRunLoopBeforeSources:
                NSLog(@"%@ 即将处理 Source", [NSThread currentThread]);
                break;
            case kCFRunLoopBeforeWaiting:
                NSLog(@"%@ 即将进入休眠", [NSThread currentThread]);
                break;
            case kCFRunLoopAfterWaiting:
                NSLog(@"%@ 从休眠中唤醒 runloop", [NSThread currentThread]);
                break;
            case kCFRunLoopExit:
                NSLog(@"%@ 即将退出 runloop ", [NSThread currentThread]);
                break;
            default:
                break;
        }
    });
    
    CFRunLoopAddObserver(CFRunLoopGetCurrent(), runLoopObserver, kCFRunLoopDefaultMode);
    
    NSRunLoop *runloop = [NSRunLoop currentRunLoop];
    [runloop addPort:[NSPort port] forMode:NSDefaultRunLoopMode];
    [runloop run];
    
    NSLog(@"%@ 不会执行到这里", [NSThread currentThread]);
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    [self performSelector:@selector(subthreadRun) onThread:self.alwasyThread withObject:nil waitUntilDone:NO];
}

- (void)subthreadRun {
    static int i = 0;
    i++;
    NSLog(@"%@ 你点击了%d次屏幕 ", [NSThread currentThread], i);
    
    if (i == 2) {
        NSLog(@"3.线程进入阻塞状态，阻塞3秒钟");
        //[NSThread sleepForTimeInterval:3.0f];
        //[NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:3.0]];
        sleep(3);
        
        NSLog(@"4.退出线程，退出线程后，该方法下面的代码不在执行");
        [NSThread exit];
        
        NSLog(@"该线程挂了");
    }
}
```
输出结果为：
```
2017-10-17 test[35026:1575126] <NSThread: 0x600000260c40>{number = 1, name = main} 1.创建线程
2017-10-17 test[35026:1575126] <NSThread: 0x600000260c40>{number = 1, name = main} 2.启动线程，包括：1）.线程进入就绪状态；2）.线程获得CPU资源后运行状态
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 该线程一直在活跃
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将进入 runloop
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将处理 Timer
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将处理 Source
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将进入休眠
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 从休眠中唤醒 runloop
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将处理 Timer
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将处理 Source
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 你点击了1次屏幕 
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将退出 runloop 
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将进入 runloop
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将处理 Timer
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将处理 Source
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将进入休眠
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 从休眠中唤醒 runloop
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将处理 Timer
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 即将处理 Source
2017-10-17 test[35026:1575230] <NSThread: 0x604000461200>{number = 3, name = (null)} 你点击了2次屏幕 
2017-10-17 test[35026:1575230] 3.线程进入阻塞状态，阻塞3秒钟
2017-10-17 test[35026:1575230] 4.退出线程，退出线程后，该方法下面的代码不在执行
```
通过输出结果得知，runloop在没有任务或事件处理时，就会进入休眠状态，当我从屏幕上点击一下，runloop就马上唤醒了，然后runloop的状态依次如下：
进入 `即将处理timer` -> `即将处理 Source` -> `处理用户事件` -> `退出runloop`，`在进入runloop` -> `即将处理Timer` -> `即将处理Source` -> `即将进入休眠`

<br/>
** 退出RunLoop的三种方式 **
- 1.当线程退出了，runloop就结束了
- 2.在运行runloop时，设置一个截止时间，如：`[self.runloop runUntilDate:[NSDate dateWithTimeIntervalSinceNow:10]];`  10秒后runloop结束了
- 3.主动调用`CFRunLoopStop(CFRunLoopRef rl)`


`NSPort` 是一个抽象类，表示通信通道，它的子类有:
- `NSMachPort` 是本地机器的端口通信，
- `NSSocketPort` 可以是本地机器，也可以远程机器的端口消息通道
- `NSMessagePort` 是一个在通信过程使用的消息类，供NSMachPort和NSSocketPort使用

<br/>
[GCD的一般认知](/2017/10/15/GCD的一般认知/)
[NSOperation的认知](/2017/10/15/NSOperation的认知/)
[iOS中的锁](/2017/10/15/iOS中的锁/)
[NSThread的认知](/2017/10/15/NSThread的认知/)
