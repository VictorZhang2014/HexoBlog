---
title: fishhook的基本使用，fishhook可以勾住系统函数
date: 2017-10-26 19:15:02
tags: Valgrind, C/C++, 内存管理
categories: iOS
---

## 1.概念
<a href="https://github.com/facebook/fishhook" target="_blank">fishhook</a>是一个非常牛的Mach-O二进制库，原因是因为它可以`动态`的`重新绑定Mach-O符号`，专门用于在iOS系统上（模拟器或者真机都可以）。

这个库的最大功能就是可以干涉系统函数，类似在Mac OS X上使用<a href="https://opensource.apple.com/source/dyld/dyld-210.2.3/include/mach-o/dyld-interposing.h" target="_blank">DYLD_INTERPOSE</a>。

在Facebook，我们惊奇的发现使用<a href="https://github.com/facebook/fishhook" target="_blank">fishhook</a>可以勾住`libSystem`的函数调用，目的了？就是用于`debugging`（又叫做`调试`）或者`tracing`（又叫做`追踪`）。
例如：对带有文件描述符的双重关闭问题进行稽核。原文是（auditing for double-close issues with file descriptors）。

## 2.用法
使用的方法也是超简单，这里了，我们就拿打开(`open()`)一个文件和关闭(`close()`)一个文件来举例。我们把系统打开文件函数`open`重写一下（其实是叫做`symbols rebinding`），也对关闭文件函数`close`重写一下（它也叫作`symbols rebinding`）。
代码如下：
```
#import <dlfcn.h>

//导入头文件
#import "fishhook.h"

//系统open函数的函数指针声明
static int (*orig_open)(const char *, int, ...);

//系统close函数的函数指针声明
static int (*orig_close)(int);

//对系统open函数重写（其实是符号重新绑定），但是我感觉就像是重写
int my_open(const char *path, int oflag, ...)
{
    va_list ap = {0};
    mode_t mode = 0;
    
    if ((oflag & O_CREAT) != 0) {
        va_start(ap, oflag);
        mode = va_arg(ap, int);
        va_end(ap);
        
        printf("1.先调用my_open()，然后在调用系统的open() ('%s', %d, %d) \n", path, oflag, mode);
        return orig_open(path, oflag, mode);
    } else {
        printf("1.先调用my_open()，然后在调用系统的open() ('%s', %d, %d) \n", path, oflag, mode);
        return orig_open(path, oflag, mode);
    }
}

//对系统close函数重写（其实是符号重新绑定），但是我感觉就像是重写
int my_close(int fd)
{
    printf("2.先调用my_close()，然后调用系统的close函数 \n");
    return orig_close(fd);
}

int main(int argc, char * argv[]) {
    @autoreleasepool {
        
        //对open和close函数的符号重新绑定
        rebind_symbols((struct rebinding[2]){{"close", my_close, (void *)&orig_close}, {"open", my_open, (void *)&orig_open}}, 2);
        
        //假设打开一个文件，并且只读取前4个字节，以后你想要操作其他的系统函数，用法也是一样的
        //调用open方法时，会先调用my_open
        int fd = open(argv[0], O_RDONLY);
        
        //读取四个字节并打印
        uint32_t magic_number = 0;
        read(fd, &magic_number, 4);
        printf("Mach-O Magic Number: %x \n", magic_number);
        
        //关闭文件描述符，会先调用my_close
        close(fd);
    
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```
输出结果为：
```
1.先调用my_open()，然后在调用系统的open() ('你的应用的路径/testFishhook.app/testFishhook', 0, 0) 
Mach-O Magic Number: feedfacf 
2.先调用my_close()，然后调用系统的close函数1.先调用my_open()，然后在调用系统的open() ('你的应用的路径/testFishhook.app/Info.plist', 0, 0) 
......
```

<br/>
** va_list, va_start, va_arg, va_end 的介绍 **
<a href="https://baike.baidu.com/item/va_list/8573665?fr=aladdin" target="_blank">百度百科</a>说: va_list是在C语言中解决变参问题的一组宏，所在头文件：#include < stdarg.h > ，用于获取不确定的个数的参数。
**用法**
- （1）首先在函数里定义一具`VA_LIST`型的变量，这个变量是指向参数的指针；
- （2）然后用`VA_START`宏初始化刚定义的`VA_LIST`变量；
- （3）然后用`VA_ARG`返回可变的参数，`VA_ARG`的第二个参数是你要返回的参数的类型（如果函数有多个可变参数的，依次调用`VA_ARG`获取各个参数）；
- （4）最后用`VA_END`宏结束可变参数的获取。

<br/>
<br/>

## 其他工具引荐介绍
Valgrind是一个相当强悍的C/C++的内存管理动态分析工具，[官方文档](http://valgrind.org/docs/manual/QuickStart.html) 。
Valgrind工具套件提供了大量的`调试`和`分析`工具，这些分析工具能帮助你并且让你的`程序更快`和`更正确`。这个超级流行的工具就叫做`Memcheck`。`Memcheck`工具能检测大量内存相关的`errors`，该工具主要针对`C`和`C++`程序，这写`errors`能引起崩溃和不可预测的行为。






