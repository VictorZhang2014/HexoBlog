---
title: CA证书的介绍与使用
date: 2017-04-01 16:29:23
tags: CA
---

## CA证书
> 1.域名型SSL证书（DV SSL）：信任等级普通，只需验证网站的真实性便可颁发证书保护网站；一般是免费的
> 2.企业型SSL证书（OV SSL）：信任等级强，须要验证企业的身份，审核严格，安全性更高；收费的
> 3.增强型SSL证书（EV SSL）：信任等级最高，一般用于银行证券等金融机构，审核严格，安全性最高，同时可以激活绿色网址栏。收费的，比OV证书贵一些
> 4.自签名证书(SelfSigned Cert) : 创建的工具有很多。如Mac中的钥匙串可以创建，IIS的服务器证书可以创建

https://help.aliyun.com/knowledge_detail/42216.html
公钥（Public Key）与私钥（Private Key）是通过一种算法得到的一个密钥对（即一个公钥和一个私钥），公钥是密钥对中公开的部分，私钥则是非公开的部分。公钥通常用于加密会话密钥、验证数字签名，或加密可以用相应的私钥解密的数据。通过这种算法得到的密钥对能保证在世界范围内是唯一的。使用这个密钥对的时候，如果用其中一个密钥加密一段数据，必须用另一个密钥解密。比如用公钥加密数据就必须用私钥解密，如果用私钥加密也必须用公钥解密，否则解密将不会成功。

## x509是数字证书的规范，P7和P12是两种封装形式。比如说同样的电影，有的是avi格式，有的是mpg。

## SSL/TLS/HTTPS的区别
SSL (Secure Socket Layer) 安全套接层  目前有三个版本1.0, 2.0, 3.0
TLS (Transport Layer Security) 传输层安全协议   是SSL的标准化后的产物，目前有1.0, 1.1, 1.2三个版本
HTTPS (Hyper Text Transfer Protocol over Secure Socket Layer) 也就是HTTP 加上 SSL/TLS

用于网站HTTPS化的SSL数字证书，当前主要分为DV SSL、OV SSL、EV SSL三种类型的证书。
symantec证书检测
https://cryptoreport.websecurity.symantec.com/checker/views/certCheck.jsp


当你的网站配置了https后，如果在IE浏览器提示证书配置错误，chrome浏览器提示不安全的连接，那就是因为IIS默认没有开启TLS，所以我们需要手动开启SSL/TLS，具体步骤如下链接
https://social.technet.microsoft.com/Forums/forefront/en-US/ec033ff6-091d-441d-8ad3-7ea411100009/ssl-with-256bit-strength
这篇链接中最重要的步骤就是
In order to get IIS7 to do 256 bit encryption, we have to ensure the cipher suit that is listed first is the following: TLS_RSA_WITH_AES_256_CBC_SHA
 
> In order to change the Cipher Suite order we can do the following:
> - Run gpedit.msc from the command line
> - within the Group Policy Object Editor, expand Computer Configuration, Administrative Templates, Network.
> - Under Network, select SSL Configuration and then double click on SSL Cipher Suite Order
> - By Default the SSL Cipher Suite Order is set to "Not Configured"
> - To enable 256-bit encryption, select the "enabled" radio button
> - Within the SSL Cipher Suites text box, remove TLS_RSA_WITH_AES_128_CBC_SHA or at least place it behind TLS_RSA_WITH_AES_256_CBC_SHA.
 
TLS_RSA_WITH_AES_256_CBC_SHA has to be the first cpiher suite listed in order for us to connect with 256-bith encryption.
 
Once the steps above have been completed, restart the server for the changes to take effect. Now we can browse a page served on the IIS7 server and confirm it is using 256 bit encryption. To do this, right click on the page, select properties and you should see the following:
TLS 1.0, AES with 256 bit encryption (High); RSA with 1024 bit exchange


iOS客户端的NSURLSession的SSL用法
```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    
    NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration delegate:self delegateQueue:[NSOperationQueue mainQueue]];
    
    NSURL *url = [NSURL URLWithString:@"https://www.domain.com/Home/LoginWithPass"];
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    [request setHTTPMethod:@"POST"];
    
    NSURLSessionDataTask *data = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        
        NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *)response;
        NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"%@ %ld",  str, (long)httpResponse.statusCode);
        
    }];
    
    [data resume];
}

/*
  摘要：适用于自签名证书，DV,OV,EV类型的证书
 
  作用：1.单向认证(One-way Authentication)
        此delegate方法会有一次调用，是NSURLAuthenticationMethodServerTrust
      2.双向认证(Bidirectional Authentication，或者Mutual Authentication，也有Two-way Authentication)
        此delegate方法会有两次调用，分别是NSURLAuthenticationMethodServerTrust和NSURLAuthenticationMethodClientCertificate

 */
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential * _Nullable))completionHandler
{
    //证书的处理方式
    NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
    
    NSURLCredential *credential = nil;
    
    //判断服务器返回的证书是否是服务器信任的
    if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) { //受信任的
        
        //获取服务器返回的证书
        credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
        
        if (credential) {
            disposition = NSURLSessionAuthChallengeUseCredential;
        } else {
            disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        }
        
    } else {
    
        //读取证书的私钥
        NSString *thePath = [[NSBundle mainBundle] pathForResource:@"webtest_ssl_Certificates" ofType:@"p12"];
        NSData *PKCS12Data = [[NSData alloc] initWithContentsOfFile:thePath];
        CFDataRef inPKCS12Data = (CFDataRef)CFBridgingRetain(PKCS12Data);
        SecIdentityRef identity = nil;
        
        //读取p12证书的私钥内容
        OSStatus result = [self extractP12Data:inPKCS12Data toIdentity:&identity];
        if(result != errSecSuccess){
            completionHandler(NSURLSessionAuthChallengeCancelAuthenticationChallenge, nil);
            return;
        }
        
        SecCertificateRef certificate = NULL;
        SecIdentityCopyCertificate(identity, &certificate);
        
        const void *certs[] = {certificate};
        CFArrayRef certArray = CFArrayCreate(kCFAllocatorDefault, certs, 1, NULL);
        
        credential = [NSURLCredential credentialWithIdentity:identity certificates:(NSArray*)CFBridgingRelease(certArray) persistence:NSURLCredentialPersistencePermanent];
        
        disposition = NSURLSessionAuthChallengeUseCredential;
    }
    
    //安装证书(即导入到iPhone的KeyChain)
    if (completionHandler) {
        completionHandler(disposition, credential);
    }
}

- (OSStatus)extractP12Data:(CFDataRef)inP12Data toIdentity:(SecIdentityRef *)identity {
    
    OSStatus securityError = errSecSuccess;
    
    CFStringRef password = CFSTR(“Your cert password”);
    const void *keys[] = { kSecImportExportPassphrase };
    const void *values[] = { password };
    
    CFDictionaryRef options = CFDictionaryCreate(NULL, keys, values, 1, NULL, NULL);
    
    CFArrayRef items = CFArrayCreate(NULL, 0, 0, NULL);
    securityError = SecPKCS12Import(inP12Data, options, &items);
    
    if (securityError == errSecSuccess) {
        CFDictionaryRef ident = CFArrayGetValueAtIndex(items, 0);
        const void *tempIdentity = NULL;
        tempIdentity = CFDictionaryGetValue(ident, kSecImportItemIdentity);
        *identity = (SecIdentityRef)tempIdentity;
    }
    
    if (options) CFRelease(options);
    if (password) CFRelease(password);
    
    return securityError;
}
```



制作证书的三种方式
1.Free https://letsencrypt.org
2.XCA介绍   http://xca.hohnstaedt.de/     
   下载地址  https://sourceforge.net/projects/xca/files/?source=navbar
3.在www.net.cn申请DV（domain verified）证书
