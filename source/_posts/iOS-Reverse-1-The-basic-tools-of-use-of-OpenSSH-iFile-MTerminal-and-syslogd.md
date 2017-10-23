---
title: iOS Reverse - (1) The basic tools of use of OpenSSH, iFile, MTerminal and
  syslogd
date: 2017-06-04 19:47:05
tags: OpenSSH, iFile, MTerminal, syslogd
---

# 1.OpenSSH

OpenSSH will install SSH service on iOS (as shown in figure below). Only 2 commands are the most commonly used: ssh is used for remote logging, scp is used for remote file transfer. The usage of ssh is as follows:

![OpenSSH](/img/iOS/ReverseEngineering/WX20170604-OpenSSH-1.png)

```
ssh user@iOSIP
```
For instance:
```
snakeninnysiMac:~ snakeninny$ ssh mobile@192.168.1.6
```
The usage of scp is as follows:
1.Copy a local file to iOS
```
scp /path/to/localFile user@iOSIP:/path/to/remoteFile
```
For instance:
```
snakeninnysiMac:~ snakeninny$ scp ~/1.png root@192.168.1.6:/var/tmp/
```
2.Copy a file from iOS to the local system
```
scp user@iOSIP:/path/to/remoteFile /path/to/localFile
```
For instance:
```
snakeninnysiMac:~ snakeninny$ scp root@192.168.1.6:/var/log/syslog ~/iOSlog
```

These two commands are relatively simple and intuitive. After installing OpenSSH, make sure to change the default login password “alpine”. There’re 2 users on iOS, i.e. root and mobile, we need to change both passwords like this:
```
FunMaker-5:~ root# passwd root Changing password for root.
New password:
Retype new password: FunMaker-5:~ root# passwd mobile Changing password for mobile. New password:
Retype new password:
```

If we forget to change the default password, there’re chances that viruses like Ikee login as root via ssh. This leads to very serious security disasters: all data on iOS including SMS, contacts, AppleID passwords and so on is at the risk of leaking, the intruder can take control over your device and do whatever he wants. Therefore, promise me you’ll change the default password after installing OpenSSH, OK?


<br/>

# 2.iFile 
iFile is a very powerful file management App, you can view it as Finder’s parallel on iOS. iFile is capable of all kinds of file operation including browsing, editing, cutting, copying and deb installing, possessing great convenience.
iFile is rather user-friendly. Before installing a deb, remember to close Cydia at first, then tap the deb file to be installed and choose “Installer” in the action sheet, as shown in figure below.

![iFile](/img/iOS/ReverseEngineering/WX20170604-iFile-1.png)
![iFile](/img/iOS/ReverseEngineering/WX20170604-iFile-2.png)

Due to iFile is chargeable. So I wrote a project which contans the most functionalities, and latter, I will finish it completely gradually, and it is free for anyone on github.
https://github.com/iOS-Reverse-Engineering-Dev/iFiler


<br/>

# 3.MTerminal
![iFile](/img/iOS/ReverseEngineering/WX20170604-MTerminal.png)

MTerminal is an open sourced Terminal on iOS with all basic functions available. The usage of MTerminal is no much difference to Terminal, if we put the screen and keyboard size aside. I
   
think the most practical scene of MTerminal is to test private methods in Cycript when we’re blanking out on the subway or something.


<br/>

# 4.syslogd to /var/log/syslog
![iFile](/img/iOS/ReverseEngineering/WX20170604-syslogd-to-1.png)
syslogd is a daemon to record system logs on iOS, and “syslogd to /var/log/syslog” is used to write the logs to a file at “/var/log/syslog”. You need to reboot iOS after you install this tweak to automatically create the file “/var/log/syslog”. This file gets larger as time goes by, you can zero clear it with the following command:

```
FunMaker-5:~ root# cat /dev/null > /var/log/syslog
```


<br/>
<br/>
<br/>
Reference : iOS App Reverse Engineering -- by snakeninny, hangcom


