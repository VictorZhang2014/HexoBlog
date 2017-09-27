---
title: 'iOS Reverse - (2) Theos introduction, installation and usage '
date: 2017-06-04 20:10:17
tags: theos, ldid
---

# 1.Introduction

Theos is a jailbreak development tool written and shared on GitHub by a friend, Dustin Howett (@DHowett). Compared with other jailbreak development tools, Theos’ greatest feature is simplicity: It’s simple to download, install, compile and publish; the built-in Logos syntax is simple to understand. It greatly reduces our work besides coding.
Additionally, iOSOpenDev, which runs as a plugin of Xcode is another frequently used tool in jailbreak development, developers who are familiar with Xcode may feel more interested in this tool, which is more integrated than Theos. But, reverse engineering deals with low-level knowledge a lot, most of the work can’t be done automatically by tools, it’d be better for you to get used to a less integrated environment. Therefore I strongly recommend Theos, when you use it to finish one practice after another, you will definitely gain a deeper understanding of iOS reverse engineering.

<br/>
# 2.Install and configure Theos
## 2.1 Install Xcode and Command Line Tools
Most iOS developers have already installed Xcode, which contains Command Line Tools. For those who don’t have Xcode yet, please download it from Mac AppStore for free. If two or more Xcodes have been installed already, one Xcode should be specified as “active” by “xcode- select”, Theos will use this Xcode by default. For example, if 3 Xcodes have been installed on your Mac, namely Xcode1.app, Xcode2.app and Xcode3.app, and you want to specify Xcode3 as active, please use the following command:

```
 snakeninnys-MacBook:~ snakeninny$ sudo xcode-select -s /Applications/Xcode3.app/Contents/Developer
```

## 2.2 Download Theos
Download Theos from GitHub using the following commands:
```
snakeninnysiMac:~ snakeninny$ export THEOS=/opt/theos
snakeninnysiMac:~ snakeninny$ sudo Git clone https://github.com/iOS-Reverse-Engineering-Dev/theos $THEOS Password:
Cloning into '/opt/theos'...
remote: Counting objects: 4116, done.
remote: Total 4116 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (4116/4116), 913.55 KiB | 15.00 KiB/s, done.
Resolving deltas: 100% (2063/2063), done.
Checking connectivity... done
```

## 2.3 Configure ldid
ldid is a tool to sign iOS executables; it replaces codesign from Xcode in jailbreak
development. Download it from http://joedj.net/ldid to “/opt/theos/bin/”, then grant it execute permission using the following command:
```
snakeninnysiMac:~ snakeninny$ sudo chmod 777 /opt/theos/bin/ldid
```

## 2.4 Configure CydiaSubstrate
First run the auto-config script in Theos:
```
snakeninnysiMac:~ snakeninny$ sudo /opt/theos/bin/bootstrap.sh substrate Password:
Bootstrapping CydiaSubstrate...
Compiling iPhoneOS CydiaSubstrate stub... default target? failed, what?
Compiling native CydiaSubstrate stub... Generating substrate.h header...
```
Here we’ll meet a bug that Theos cannot generate a working libsubstrate.dylib, which requires our manual fixes. Piece of cake: first search and install CydiaSubstrate in Cydia, as shown in figure below.

![CydiaSubstrate](/img/WX20170604-cydiasubstrate-1.png)

Then copy “/Library/Frameworks/CydiaSubstrate.framework/CydiaSubstrate” on iOS to somewhere on OSX such as the desktop using iFunBox or scp. Rename it libsubstrate.dylib and copy it to “/opt/theos/lib/libsubstrate.dylib” to replace the invalid file.

## 2.5 Configure dpkg-deb
The standard installation package format in jailbreak development is deb, which can be
manipulated by dpkg-deb. Theos uses dpkg-deb to pack projects to debs. Download dm.pl from
https://raw.githubusercontent.com/DHowett/dm.pl/master/dm.pl, rename it dpkg-deb and move it to “/opt/theos/bin/”, then grant it execute permission using the following command:
```
snakeninnysiMac:~ snakeninny$ sudo chmod 777 /opt/theos/bin/dpkg-deb
```

## 2.6 Configure Theos NIC templates
It is convenient for us to create various Theos projects because Theos NIC templates have 5
different Theos project templates. You can also get 5 extra templates from https://github.com/DHowett/theos-nic-templates/archive/master.zip and put the 5 extracted .tar files under “/opt/theos/templates/iphone/”. Some default values of NIC can be customized, please refer to http://iphonedevwiki.net/index.php/NIC#How_to_set_default_values.

There are extra 5 templates here.  https://github.com/DHowett/theos-nic-templates/archive/master.zip
Copy the extra 5 templates to `/opt/theos/templates/iphone/`



<br/>
# 3. The use of theos
## 3.1 Create Theos project
Change Theos’ working directory to whatever you want (like mine is “/User/VictorZhang/Documents/iOS/Projects/theos/”), and then enter `/opt/theos/bin/nic.pl` to start NIC (New Instance Creator), as follows:
![theos](/img/theos-1.png)

## 3.2 A simple theos project
That is t will alert and UIAlertView when every launching; Here we select nineth template
![theos](/img/thoes-2.png)

```
1)  Chose “9” to create a tweak project:
>Choose a Template (required): 9
2)  Enter the name of the tweak project: 
>Project Name (required): iOSREGreetings
3)  Enter a bundle identifier as the name of the deb package:
Package Name [com.yourcompany.iosreproject]: com.iosre.iosregreetings
4)  Enter the name of the tweak author:
>Author/Maintainer Name [snakeninny]: Victor
5)  Enter the bundle identifier, the bundle identifier deponds on what process/application you want to hook/tweak, here I take SpringBoard for instance, so the bundle identifier as `com.apple.springboard`
>[iphone/tweak] “MobileSubstrate Bundle filter [com.apple.springboard]: com.apple.springboard

```
After these 5 simple steps, a folder named iosregreetings is created in the current directory, which contains the tweak project we just created.
Then effect 1
![theos](/img/theos-3.png)


## 3.3 Modify Makefile file
The project files, frameworks and libraries are all specified in Makefile, making the whole compile process automatic. The Makefile of iOSREGreetings is shown as follows:
```
export THEOS_DEVICE_IP = 10.18.136.168   ## The specifies IP's device will be installed this project

export ARCHS = armv7 arm64           ## Specify CPU architectures. Different CPU architectures should be separated by spaces in the above configuration. Note, Apps with arm64 instructions are not compatible with armv7/armv7s dylibs, they have to link dylibs of arm64. In the vast majority of cases, just leave it as “armv7 arm64”.

export TARGET = iphone:clang:latest:8.0   ## Specify the SDK version

include theos/makefiles/common.mk     ## This is a fixed writing pattern, don’t make changes.

TWEAK_NAME = iOSReGreetings          ## The tweak name, i.e. the “Project name” in NIC when we create a Theos project. It corresponds to the “Name” field of the control file, please don’t change it.

iOSReGreetings_FILES = Tweak.xm      ## Source files of the tweak project, excluding headers; multiple files should be separated by spaces, like: `iOSREProject_FILES = Tweak.xm Hook.xm New.x ObjC.m ObjC++.mm`

iOSReGreetings_FRAMEWORKS = UIKit    ## Import frameworks. There is nothing to explain. However, as tweak developers, how to import private frameworks attracts us more for sure. It’s not much difference to importing documented frameworks: `iOSREProject_PRIVATE_FRAMEWORKS = AppSupport ChatKit IMCore`


include $(THEOS_MAKE_PATH)/tweak.mk        ## According to different types of Theos projects, different .mk files will be included. In the beginning stage of iOS reverse engineering, 3 types of projects are commonly created, they are Application, Tweak and Tool, whose corresponding files are application.mk, tweak.mk and tool.mk respectively. It can be changed on demand.

after-install::
    install.exec "killall -9 SpringBoard"    ## I guess you know what’s the purpose of these two lines of code from the literal meaning, which is to kill SpringBoard after the tweak is installed during development, and to let CydiaSubstrate load the proper dylibs into SpringBoard when it relaunches.
```

 
## 3.4 Modify Tweak.xm file
```
%hook SpringBoard
- (void)applicationDidFinishLaunching:(id)application {
   %orig;
   UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Come to http://bbs.iosre.com for more fun!" message:nil delegate:self cancelButtonTitle:@"OK" otherButtonTitles:nil];
   [alert show];
   [alert release]; }
%end
```
Tweak.xm file is written by Logos language. Full Explanation of Logos http://iphonedevwiki.net/index.php/Logos?spm=5176.100239.blogcont60056.5.JsFRmA
The commonly usage as shown below
```
        %hook Indicates that what class name you want to hook ,with ending of %end
        %log Prints logs to `/var/log/syslogd` 
        %orig Performs the hooked original method, like OC's syntax : [super method]
        %group Indicates that %group groups many of %hook , with ending of %end
        %init Indicates that it is initialized a certain %group, and it works after initialized, and %init has to be called in a %hook
        %ctor It is tweak's constructor，if you do not call it, theos will automatically generate a %ctor and call %init(_ungrouped) implicitly or explicitly, i.e. %ctor { %init(_ungrouped)}
        %new Indicates that the directive is to add new method to a class, just like class_addMethod in OC Runtime
        %c Indicates that the directive is trying to get a class name of string formatted, just like objc_getClass in OC Runtime。
```



## 3.5 Modify control file
```
Package: com.victor.iosregreetings
Name: iOSReGreetings
Depends: mobilesubstrate, firmware (>= 8.0)
Version: 1.0
Architecture: iphoneos-arm
Description: This is my first tweak project , very simple!
Maintainer: Victor
Author: Victor
Section: Tweaks
Homepage: http://www.googleplust.party
```

The control file contains the basis of infomations of deb package manager, including: Package Identifier, Project Name, Depends, Version, Architecture, Description, Author and Homepage etc.


## 3.6 Make, package and install
```
cd iOSReGreetings/
make package install
```

`make package install` is a combination command of  `make` , `make package` and `make install`

While installing , the Terminal will ask for twice SSH password, don't worry, it's normal conditions.
When installed completely, iPhone will launch automatically. 
NOTE THAT: Do not unlock your iPhone, just press home button once so that the screen will light, and you'll see the figure as shown below.
![theos](/img/theos-4-1.png)

And then , unlock your iPhone, open Cydian, you'll see the iOSREGreetings app as shown in figure below.
![theos](/img/theos-5-1.png)


## 3.7 clean command

> make clean     # It will clean the current directory's packaged files
> rm *.deb       # It will remove the current directory's *.deb packages
 


