---
title: iOS App Dumps Encrypted Shell and Disassembling
date: 2017-04-15 19:36:52
tags: iOS Reverse Engineering, Unshell, Disassembling
---

## Overview
Software reverse engineering refers to the process of deducing the implementation and design details of a program or a system by analyzing the functions, structures or behaviors of it. When we are very interested in a certain software feature while not having the access to the source code, we can try to analyze it by reverse engineering.

Although the recipe of Coca-Cola is highly confidential, some other companies can still copy its taste. Although we don't have access to the source code of others' Apps, we can dig into their details of reverse engineering.

## PDF Document
1. [FOR MORE DETAILS OF iOS REVERSE ENGINEERING, SEE HERE.](https://drive.google.com/file/d/0B9vGMCW0Vvs3UHd0b0UwQ2x1OUk/view?usp=sharing)
2. [iOS应用逆向工程（第二版）](http://download.csdn.net/detail/u013538542/9815713)

## Chinese Blog
[iOS逆向 - 获取AppStore上的应用的所有头文件和源文件  http://blog.csdn.net/u013538542/article/details/70196590](http://blog.csdn.net/u013538542/article/details/70196590)


## What's the main purpose of this article saying?
>1.Browses all directories on the iPhone;
>2.SSH login to the jailbreak iPhone;
>3.Dumps the encrypted App which is downloaded from the App Store and shows you a decrypted Mach-O file(It indicates the binary file of the IPA);
>4.Gets all header files and disassembles the binary file of the App, shows you the pseudo code of Objective-C;
>5.Copies files from Mac to iPhone, vice versa.

## Let's get prepared!
First of all, you have to prepare some tools and libraries which I show you underneath.
#### Hardware
>1.A jailbroken iPhone/iPad/iPod;
>2.A MacBook Pro.

#### Software
>1.[Cydia](https://en.wikipedia.org/wiki/Cydia) has been installed in the iPhone;
>2.Installs OpenSSH, directly search it in Cydia in the iPhone;
>3.Download the repository of [dumpdecrypted](https://github.com/VictorZhang2014/dumpdecrypted), which unshells the app;
>4.[iFunBox](http://www.i-funbox.com/) which can browse all directories and files on your iPhone;
>5.[class-dump](http://stevenygard.com/projects/class-dump/) is capable of getting all header files of an application;
>6.[Hopper](https://www.hopperapp.com/) is capable of analyse the source code of an application in pseudo code. If you'd like to use [IDA](https://www.hex-rays.com/products/ida/) that's okay for it.

## Let's get started!
When you get ready for these necessary software and hardware, then I start with [dumpdecrypted](https://github.com/VictorZhang2014/dumpdecrypted) is open source project that you can find it on [github]((https://github.com/VictorZhang2014/dumpdecrypted)).

### 1.Firstly, we have to generate a dumpdecrypted.dylib that is a dynamic library, because we need it for unshelling the encrypted binary file.
![dumpdecrypted command](/img/dumpdecryptedCommand.png)
You'll see a dynamic library
![dumpdecrypted directory](/img/dumpdecryptedDirectory.png)


<br/>
<br/>
### 2.Secondly, we use [iFunBox](http://www.i-funbox.com/) or OpenSSH connecting to the jailbroken device, and find applications.
Here, I choose [iFunBox](http://www.i-funbox.com/).

>Figure - 1. All the sandbox of applications directory, but you have to find one what you want.

![iFunBox Applications SandBox Directory](/img/iFunBoxApplicationsSandBoxDir.png)

>Figure - 2. Application Directory, we take WeChat.app for instance.

![iFunBox Application Directory](/img/iFunBoxApplicationDirectory.png)

>Figure - 3. The SandBox of WeChat application  [What's Sandbox on iPhone?](https://developer.apple.com/library/content/documentation/Security/Conceptual/AppSandboxDesignGuide/AboutAppSandbox/AboutAppSandbox.html)

![iFunBox Application SandBox Directory](/img/iFunBoxApplicationSandBoxDirectory.png)

>Figure - 4. Copies dumpdecrypted.dylib to the document directory of sandbox of WeChat application.
![iFunBox ApplicationSandBox dumpdecrypted.dylib](/img/iFunBoxApplicationSandBoxDumpDecrypted.png)

<br/>
>Please Note: If you don't know how to find the application out you want, then use of Terminal on your Mac.  You connect to the Jailbroken iPhone by OpenSSH. When you will have connected to the iPhone, use of `find` command to search the application name and its path will show you directly.


<br/>
<br/>
### 3.The stirring moment will has showcased underneath. That unshell the application. One thing we all knew that the application which had been encrypted its binary file while we submitted it to the App Store. We download any one of application to the iPhone from App Store so that we can analyse it.

1.At first, Go to the sandbox of the application use of OpenSSH. You'll see the `dumpdecrypted.dylib` displayed here;
![JailBrokeniPhoneSandBoxWeChatDocuments](/img/JailBrokeniPhoneSandBoxWeChatDocuments.png)

2.Then, we are going to dump the encrypted binary of WeChat.
```
DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib /private/var/mobile/Containers/Bundle/Application/EBB2963F-EB9D-4BE1-9D9C-4A281C0D827E/WeChat.app/WeChat
```
![JailBrokeniPhoneWeChatDumpEncryptedBinary](/img/JailBrokeniPhoneWeChatDumpEncryptedBinary.png)

3.Finally, we got `WeChat.decrypted` file which is meant for no encrypted, no secure in the WeChat sandbox Documents directory.
![iFunBoxWeChatDumppedByDumpDecryptedTool](/img/iFunBoxWeChatDumppedByDumpDecryptedTool.png) 


<br/>
<br/>
### 4.Export all header files of WeChat application by `class-dump`
Copies `WeChat.decrypted` file from the Jailbroken iPhone to your Mac use by iFunBox.

1.Normally, we use the following command to get all header files, bu it doesn't work.
```
class-dump -S -s -H WeChat.decrypted -o Headers/
```
So, I've found a solution for this circumstance. Run following command 
```
class-dump --arch armv7 -S -s -H WeChat.decrypted -o Headers/
```
Aha, it works! Really!!
See? All header files of WeChat application displays here.
![WeChatLoginWithApiAllHeaderFiles](/img/WeChatLoginWithApiAllHeaderFiles.png)

I opened a class `WTLoginApi`
![WeChatLoginWithApiHeaderFileByHopper](/img/WeChatLoginWithApiHeaderFileByHopper.png)


<br/>
<br/>
### 5. Lookup source code use by Hopper
We have seen all header files of WeChat application, even if it was downloaded from the App Store,but our goal is to find more source out, isn't it?
So, I'm bound to use Hopper to find source code I want. 
Why Hopper? Actually, I'm prone to use Hopper, instead of IDA. If you're prone to use IDA, that's a good option.

We take class `WTLoginApi` for example, and its function `loginWithPasswd:andPasswd:andSigBitmap:andLoginFlag:retData:`. See below.
![WeChatLoginWithApiByHopper](/img/WeChatLoginWithApiByHopper.png)


As a matter of fact, if you want to understand it, you have to know about Assembly Language. Hopper gave us a very clear source code and call track, for learning reverse engineering, try learning AL.






