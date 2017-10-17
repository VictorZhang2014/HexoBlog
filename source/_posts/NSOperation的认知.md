---
title: NSOperation的认知
date: 2017-10-15 17:28:29
tags: iOS, NSOperation
---

[【官方文档】](https://developer.apple.com/documentation/foundation/nsoperation?language=objc)

# NSOperation
## 目录
### 1.NSOperation简介
### 2.NSOperation和NSOperationQueue的基本使用
####  2.1 创建任务
####  2.2 创建队列
####  2.3 将任务添加到队列中
### 3.操作依赖
### 4.一些其他方法

<br/>
### **1.NSOperation简介**
`NSOperation`是`Apple`提供给开发者的一套`多线程`解决方案，实际上是基于GCD的一套更`高级封装`，完全Objective-C代码。`简单`、`易用`、代码`可读性高`。

NSOperation需要配合NSOperationQueue来实现多线程，因为默认情况下
- NSOperation`单独使用`时是系统`同步`执行`操作`，并`没有`开启`新线程`的能力，只有配合NSOperationQueue才能实现异步执行

因为NSOperation是基于GCD的，那么`使用`起来也和`GCD差不多`，其中，`NSOperation`相当于`GCD`中的`任务`，而`NSOperationQueue`则相当于GCD中的`队列`。
NSOperation实现多线程的使用步骤分为三步：

- 1.创建任务：先将需要执行的操作封装到一个NSOperation对象中
- 2.创建队列：创建NSOperationQueue对象
- 3.将任务加入到队列中，然后将NSOperation对象加入到NSOperationQueue中，之后，系统就会从Queue中读取出来，在新线程中执行操作。

以下我们来看下`NSOperation`和`NSOperationQueue`的基本使用

<br/>
### **2.NSOperation和NSOperationQueue的基本使用**
NSOperation是一个抽象类，不能封装任务，我们只有使用它的子类来封装任务。有三种方式来封装任务，如下：
- 1.使用子类NSInvocationOperation
- 2.使用子类NSBlockOperation
- 3.自定义一个类派生自NSOperation，定义一些相应的方法

<br/>
####  **2.1 创建任务**
比如：我们先不使用NSOperationQueue，而是单独使用`NSInvocationOperation`和`NSBlockOperation`，分别如下：

** 2.1.1 NSInvocationOperation **
```
- (void)invocationOp {
    NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
    [op start];
}

- (void)run {
    NSLog(@"%@", [NSThread currentThread]);
}
```
输出结果如下，证明了单独使用`NSInvocationOperation`时其实是在主线程中执行，并没有开启新线程。
```
2017-10-15 17:43:57.044045+0800 test[8700:498048] <NSThread: 0x600000074a40>{number = 1, name = main}
```

** 2.1.2 NSBlockOperation **
```
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"%@", [NSThread currentThread]);
    }];
    [op start];
```
输出结果如下，同样地，`NSBlockOperation`实际也是在主线程执行的，没有开启新线程。
```
2017-10-15 17:45:47.120285+0800 test[8760:499896] <NSThread: 0x604000060340>{number = 1, name = main}
```

`NSBlockOperation`还提供一个方法 `addExecutionBlock`，通过该方法添加的`block代码块`就是在子线程中运行的
```
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        // 在主线程
        NSLog(@"1------%@", [NSThread currentThread]);
    }];
    
    //添加额外的任务(在子线程执行)
    [op addExecutionBlock:^{
        NSLog(@"2------%@", [NSThread currentThread]);
    }];
    [op addExecutionBlock:^{
        NSLog(@"3------%@", [NSThread currentThread]);
    }];
    [op addExecutionBlock:^{
        NSLog(@"4------%@", [NSThread currentThread]);
    }];
    [op addExecutionBlock:^{
        NSLog(@"5------%@", [NSThread currentThread]);
    }];
    [op start];
```
输出结果如下，`addExecutionBlock:`会开启子线程来执行任务，而`blockOperationWithBlock:`依旧是在主线程中执行任务的， 只是执行顺序会不一致
```
2017-10-15 17:46:51.401891+0800 test[8801:501346] 2------<NSThread: 0x600000068440>{number = 3, name = (null)}
2017-10-15 17:46:51.401891+0800 test[8801:501045] 1------<NSThread: 0x604000069040>{number = 1, name = main}
2017-10-15 17:46:51.401894+0800 test[8801:501347] 3------<NSThread: 0x6000002621c0>{number = 4, name = (null)}
2017-10-15 17:46:51.401891+0800 test[8801:501348] 4------<NSThread: 0x60400027e100>{number = 5, name = (null)}
2017-10-15 17:46:51.402125+0800 test[8801:501346] 5------<NSThread: 0x600000068440>{number = 3, name = (null)}
```

** 2.1.3 自定义一个类，派生自NSOperation **
```
@interface ZQRunOperation : NSOperation

@end

@implementation ZQRunOperation

- (void)main {
    NSLog(@"ZQRunOperation类 --- %@", [NSThread currentThread]);
}

@end
```
调用
```
    ZQRunOperation *myOp = [[ZQRunOperation alloc] init];
    [myOp start];
```
输出
```
2017-10-15 18:15:34.270387+0800 test[9660:515849] ZQRunOperation类 --- <NSThread: 0x6040002619c0>{number = 1, name = main}
```
`自定义`的类，根据`你的需要`，可以派生自`NSInvocationOperation`或者`NSBlockOperation`


<br/>
####  **2.2 创建队列**
使用`NSOperationQueue`和GCD的并发队列和串行队列有一点不同，是：
`NSOperationQueue`一共有两种队列，分别是：`主队列`和`其他队列`；其中其它队列就包含了串行和并发。

串行和并发执行的关键点，主要根据`maxConcurrentOperationCount`参数来`区分`，这个参数的意思是`最大并发数`
- 1.默认情况下`maxConcurrentOperationCount` 为-1，表示不进行限制，也就是并发执行
- 2.当`maxConcurrentOperationCount`设置为1时，就表示串行执行
- 3.当`maxConcurrentOperationCount`设置为大于1，就表示并发执行，加入程序员设置的值大于系统并发的最大值，那么系统也会根据情况自动调整的

**声明主队列**
```
NSOperationQueue *queue = [NSOperationQueue mainQueue];
```
把任务添加到变量`queue`中，就表示所有的任务都是在主队列中执行

**其它队列**
```
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
```
把任务添加到此变量`queue`中，就表示所有的任务会在子线程中执行，是串行执行还是并发执行取决于上面提到的参数`maxConcurrentOperationCount`

<br/>
####  **2.3 将任务添加到队列中**
接下来，我们就需要把任务添加到队列中了，使用方法 `addOperation:`，如下代码所示：
```
- (void)queue {
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
    NSInvocationOperation *invocationOp = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
    
    NSBlockOperation *blockOp = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            NSLog(@"NSBlockOperation %@", [NSThread currentThread]);
        }
    }];
    [queue addOperation:invocationOp];
    [queue addOperation:blockOp];
}

- (void)run {
    for (int i = 0; i < 2; i++) {
        NSLog(@"NSInvocationOperation %@", [NSThread currentThread]);
    }
}

```
输出结果如下，得知两点：一是，任务在`子线程`中`执行`的，二是，任务是`并行执行`的
```
2017-10-15 test[10378:538549] NSBlockOperation <NSThread: 0x600000465000>{number = 3, name = (null)}
2017-10-15 test[10378:538551] NSInvocationOperation <NSThread: 0x600000464ec0>{number = 4, name = (null)}
2017-10-15 test[10378:538549] NSBlockOperation <NSThread: 0x600000465000>{number = 3, name = (null)}
2017-10-15 test[10378:538551] NSInvocationOperation <NSThread: 0x600000464ec0>{number = 4, name = (null)}
```

还有一种方式是，直接给`NSOperationQueue`添加block任务 使用方法 `addOperationWithBlock:`
```
- (void)queue {
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            NSLog(@"NSOperationQueue直接添加block任务 %@", [NSThread currentThread]);
        }
    }];
    
    NSInvocationOperation *invocationOp = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
    
    NSBlockOperation *blockOp = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            NSLog(@"NSBlockOperation %@", [NSThread currentThread]);
        }
    }];
    [queue addOperation:invocationOp];
    [queue addOperation:blockOp];
}

- (void)run {
    for (int i = 0; i < 2; i++) {
        NSLog(@"NSInvocationOperation %@", [NSThread currentThread]);
    }
}
```
输出如下，得知，这也是在子线程中执行的，也是并发的
```
2017-10-15 [10629:542729] NSOperationQueue直接添加block任务 <NSThread: 0x60000026f880>
2017-10-15 [10629:542728] NSBlockOperation <NSThread: 0x6040004630c0>{number = 4, name = 
2017-10-15 [10629:542730] NSInvocationOperation <NSThread: 0x6000000719c0>{number = 5, 
2017-10-15 [10629:542728] NSBlockOperation <NSThread: 0x6040004630c0>{number = 4, name = 
2017-10-15 [10629:542729] NSOperationQueue直接添加block任务 <NSThread: 0x60000026f880>
2017-10-15 [10629:542730] NSInvocationOperation <NSThread: 0x6000000719c0>{number = 5, name = (null)}
```

上面说的几种方式都是`NSOperationQueue`的`并行队列`执行的，下面来一个`串行队列`的例子
```
- (void)queue {
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    queue.maxConcurrentOperationCount = 1;//设置为1，就表示串行队列
    
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            NSLog(@"NSOperationQueue直接添加block任务 %@", [NSThread currentThread]);
        }
    }];
    
    NSInvocationOperation *invocationOp = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
    
    NSBlockOperation *blockOp = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            NSLog(@"NSBlockOperation %@", [NSThread currentThread]);
        }
    }];
    [queue addOperation:invocationOp];
    [queue addOperation:blockOp];
}

- (void)run {
    for (int i = 0; i < 2; i++) {
        NSLog(@"NSInvocationOperation %@", [NSThread currentThread]);
    }
}
```
输出结果如下，从结果可以看出，所有的任务都是依次执行的，即串行队列执行任务
```
2017-10-15 test[10749:544638] NSOperationQueue直接添加block任务 <NSThread: 0x6000002713c0>{number = 3, name = (null)}
2017-10-15 test[10749:544638] NSOperationQueue直接添加block任务 <NSThread: 0x6000002713c0>{number = 3, name = (null)}
2017-10-15 test[10749:544638] NSInvocationOperation <NSThread: 0x6000002713c0>{number = 3, name = (null)}
2017-10-15 test[10749:544638] NSInvocationOperation <NSThread: 0x6000002713c0>{number = 3, name = (null)}
2017-10-15 test[10749:544638] NSBlockOperation <NSThread: 0x6000002713c0>{number = 3, name = (null)}
2017-10-15 test[10749:544638] NSBlockOperation <NSThread: 0x6000002713c0>{number = 3, name = (null)}

```


<br/>
### **3.操作依赖**
`NSOperation`和`NSOperationQueue`最吸引人的地方是它能添加操作之间的依赖关系。
比如：A, B, C三个任务操作，根据依赖关系，任务的执行顺序就不同，如下代码所示：

```
- (void)addDependenciesOperations {
    //创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
    //创建任务
    NSBlockOperation *opA = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"NSBlockOperation A  %@", [NSThread currentThread]);
    }];
    NSBlockOperation *opB = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"NSBlockOperation B  %@", [NSThread currentThread]);
    }];
    NSInvocationOperation *opC = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
    
    //添加依赖
    [opB addDependency:opC]; //opB依赖于opC
    [opA addDependency:opC]; //opA依赖于opC
    [opA addDependency:opB]; //opA依赖于opB
    //所以执行顺序应该是 opC -> opB -> opA
    
    //添加任务
    [queue addOperation:opA];
    [queue addOperation:opB];
    [queue addOperation:opC];
}

- (void)run {
    for (int i = 0; i < 2; i++) {
        NSLog(@"NSInvocationOperation C %@", [NSThread currentThread]);
    }
}
```
输出如下，得知，设置了依赖，就可以说是串行队列执行任务了
```
2017-10-15 test[10979:551447] NSInvocationOperation C <NSThread: 0x600000267080>{number = 3, name = (null)}
2017-10-15 test[10979:551447] NSInvocationOperation C <NSThread: 0x600000267080>{number = 3, name = (null)}
2017-10-15 test[10979:551448] NSBlockOperation B  <NSThread: 0x600000267200>{number = 4, name = (null)}
2017-10-15 test[10979:551447] NSBlockOperation A  <NSThread: 0x600000267080>{number = 3, name = (null)}
```
当然了，添加的依赖不一定要三个，一个也可以，如下
```
    //添加依赖
    [opB addDependency:opC]; //opB依赖于opC
    //所以任务执行顺序应该是 opA -> opC -> opB
```


<br/>
### **4.一些其他方法**
- `- (void)cancel;`  NSOperation提供的取消方法，可以取消单个操作
- `- (void)cancelAllOperations;`  NSOperationQueue提供的取消队列里所有的任务的方法
- `- (void)setSuspended:(BOOL)b;`  可以设置任务的暂停与恢复，YES表示暂停队列任务，NO表示恢复队列执行
- `- (BOOL)isSuspended;` 判断暂停状态

<br/>
*** 注意 ***
* 暂停和取消的区别在于：暂停操作后，还可以恢复操作，继续向下执行；而取消操作之后，所有的操作，再也恢复不了了，而且剩下的任务也都将取消掉了

<br/>
[GCD的一般认知，打开](/2017/10/15/GCD的一般认知/)
[iOS中的锁，打开](/2017/10/15/iOS中的锁/)
[NSThread的认知，打开](/2017/10/15/NSThread的认知/)
[iOS的Runloop认知，打开](/2017/10/15/iOS的Runloop认知/)

