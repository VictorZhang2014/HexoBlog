---
title: 'RSA Example in iOS/C#'
date: 2017-03-31 18:39:55
tags: RSA, Cryptography
categories: iOS
---

# RSA Cryptography
A set of RSA Cryptographic ways to your project in iOS/C#
[ZRCryptographyOC](https://github.com/VictorZhang2014/ZRCryptographyOC)

## RSA Overview
RSA is an algorithm used by modern computers to encrypt and decrypt messages. It is an asymetric cryptographic algorithm. Asymetric means that there are two different keys. This is also called public key cryptography, because one of them can be given to everyone. The other key must be kept private. It is based on the fact that finding the factors of an integer is hard(the factoring problem). RSA stands for Ron Rivest, Adi Shamir and Leonard Adleman, who first publicly described it in 1978. A user of RSA creates and then publishes the product of two large prime numbers, along with an auxiliary value, as their public key. The prime factors must be kept secret. Anyone can use the public key to encrypt a message, but with currently published methods, if the public key is large enough, only someone with knowledge of the prime factors can feasibly decode the message.

[See more infomation](https://simple.wikipedia.org/wiki/RSA_(algorithm))

#### Please Note
> The public key to encrypt messages, after encrypted, it must be encoded by base 64.
> The private key to decrypt messages, before decrypting, it must be decoded by base 64.

## How do I create a public key and a private key? See blow.
First of all, open up the Terminal in your MAC.

1.Generates a private key certificate with extension .pem
```
> openssl genrsa -out private_key.pem 1024
```

2.Generates a certificate file with extension .csr
```
> openssl req -new -key private_key.pem -out rsaCertReq.csr
```
Then, you have to enter extra information by the hint on the terminal of your MAC.

3.Generates a certificate file with extension .crt using by x509
```
> openssl x509 -req -days 3650 -in rsaCertReq.csr -signkey private_key.pem -out rsaCert.crt
```

4.Generates a public key certificate with extension .der
```
> openssl x509 -outform der -in rsaCert.crt -out public_key.der 
```

5. Generates a private key certificate with extension .p12, it will be entered password by the hint
```
> openssl pkcs12 -export -out private_key.p12 -inkey private_key.pem -in rsaCert.crt
```

So, we have a public key certificate `public_key.der` and a private key certificate `private_key.p12`.


## How do I use these two certificate? See below.
The code was written in Objective-C in iOS client.
1.Invokes by main function
```
#import <Foundation/Foundation.h>

//Download here https://github.com/VictorZhang2014/ZRCryptographyOC
#import <ZRCryptographyOC/ZRCryptographyOC.h>

/**************** The following methods are decryption/encryption by using a certificate   ****************/
NSString * EncryptoString(NSString *inputStr)
{
    ZRCryptographyOC *rsa = [[ZRCryptographyOC alloc] init];
    
    //Load your public Key file
    [rsa loadPublicKeyFromFile:@"/Your bundle path/public_key.der"];
    
    //encrypt
    return [rsa rsaEncryptString:inputStr];
}

NSString *DecryptoString(NSString *secureText)
{
    ZRCryptographyOC *rsa = [[ZRCryptographyOC alloc] init];
    
    //Load your private key file
    [rsa loadPrivateKeyFromFile:@"/Your bundle path/private_key.p12" password:@"123456"];
    
    //decrypt
    return [rsa rsaDecryptString:secureText];
}

void TestRSACryptography()
{
    NSLog(@"\n\n\n\n\n");
    
    //1.Encrypt String
    NSString *hello = @"RSA is an algorithm used by modern computers to encrypt and decrypt messages. It is an asymmetric cryptographic algorithm. Asymmetric means that there are two different keys. This is also called public key cryptography, because one of them can be given to everyone. The other key must be kept private. It is based on the fact that finding the factors of an integer is hard (the factoring problem). RSA stands for Ron Rivest, Adi Shamir and Leonard Adleman, who first publicly described it in 1978. A user of RSA creates and then publishes the product of two large prime numbers, along with an auxiliary value, as their public key. The prime factors must be kept secret. Anyone can use the public key to encrypt a message, but with currently published methods, if the public key is large enough, only someone with knowledge of the prime factors can feasibly decode the message.";
    
    NSString *secureHello = EncryptoString(hello);
    NSLog(@"Cipher Data : %@ \n", secureHello);
    
    //2.Decrypt String
    NSString *decryptStr = DecryptoString(secureHello);
    NSLog(@"Decrypted Data: %@", decryptStr);
}
``` 

#### A best lib for encrytion and decryption
[ZRCryptographyOC](https://github.com/VictorZhang2014/ZRCryptographyOC)

## But how interact among iOS client and C# Server?
There are several ways that I show you.
1.Client has public key;
2.Server has private key, and server can get public key in terms of private key;
3.Client send data (which will encrypt by public key) to server.
4.Server receive data and decrypt it by private key; And server identifies whether the data is trusted from client, if yes, then sends its private key to client, if not, refuses respond;
5.If the client is trusted and it receives private key; Next time, server sends data to client that can decrypt by the private key.


## Please Note
> If you have any problem about this, commits or suggests below. Thank you for reading.

