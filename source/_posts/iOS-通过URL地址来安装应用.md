---
title: iOS 通过URL地址来安装应用
date: 2017-04-01 16:30:54
tags: iOS app installation
---

## iOS应用通过URL地址来安装

## 完整下载地址
```
itms-services://?action=download-manifest&url=https://www.domain.cn/d/up/package/my.application.ipa.plist
```

## 其中plist文件具体内容是

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>items</key>
        <array>
                <dict>
                        <key>assets</key>
                        <array>
                                <dict>
                                        <key>kind</key>
                                        <string>software-package</string>
                                        <key>url</key>
                                        <string>https://www.yourdomain.cn/download/2.1.0-10637/test.116d7.56d82df.20170122.test.domain.cn.ipa.plist</string>
                                </dict>
                                <dict>
                                        <key>kind</key>
                                        <string>full-size-image</string>
                                        <key>needs-shine</key>
                                        <true/>
                                        <key>url</key>
                                        <string>http://www.yourdomain.cn/d/images/logo_full.png</string>
                                </dict>
                                <dict>
                                        <key>kind</key>
                                        <string>display-image</string>
                                        <key>needs-shine</key>
                                        <true/>
                                        <key>url</key>
                                        <string>http://www.yourdomain.cn/d/images/logo_icon.png</string>
                                </dict>
                        </array>
                        <key>metadata</key>
                        <dict>
                               <key>bundle-identifier</key>
                                <string>com.company.test</string>
                                <key>bundle-version</key>
                                <string>2.1.0</string>
                                <key>kind</key>
                                <string>software</string>
                                <key>title</key>
                                <string>我的应用</string>
                        </dict>
                </dict>
        </array>
</dict>
</plist>
```
