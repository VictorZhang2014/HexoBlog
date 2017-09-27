---
title: 'PushNotification in C#,Objective-c/C'
date: 2017-01-18 17:46:50
tags: C#,APNs,Objective-C,C
---

# PushNotification
A simplest way to push notification on your own server in C# and C/Objective-C.


## APNs Overview
Apple Push Notification service (APNs) is the centerpiece of the remote notifications feature. It is a robust and highly efficient service for propagating information to iOS (and, indirectly, WatchOS), tvOS, and macOS devices. On initial launch, your app establishes an accredited and encrypted IP connection with APNs from the user's device. Over time, APNs delivers notifications using this persistent connection. If a notification arrives when your app is not running. the device receives the notification and handles its delivery to your app at an appropriate time.


In addition to APNs and your app, another piece is required for the delivery of remote notifications. You must configure your own server to originate those notifications. Your server, known as the provider, has the following responsibilities:
* It receives device tokens and relevant data from your app.
* It determines when remote notifications need to be sent to a device.
* It communicates the notification data to APNs, which then handles the delivery of the notifications to that device.

For each remote notification, your provider:
* Constructs a JSON dictionary with the notification’s payload; described in the Remote Notification Payload.
* Attaches the payload and an appropriate device token to an HTTP/2 request.
* Sends the request to APNs over a persistent and secure channel that uses the HTTP/2 network protocol.


![Notification Flow](/img/remote_notif_simple_2x.png)


## What can this repository do?
- Provide the simplest way to send remote notification to user's device in C#
- Using the most simple code in C#/iOS , a newcomer can be able to understood
- In order to test your iOS app when you're an iOS developer
- In order to write the code of server end when you're an ASP.NET/C# developer

For more details, linked to [here](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1 "APNs").



## Introduction
Let me introduce the server end(C#) at first.
See the flow briefly firstly.
- 1.Using `TcpClient` and `SslStream` classes to connect with Apple Server
- 2.Setting certificate through `AuthenticateAsClient()` method within a passphrase, this method involves setting basic parameters and handshaking
- 3.`Write()` for writing data bytes to APNs before splicing a whole string of payload

<br/>

- Here is the simplest `payload` of a remote notification
```
{"aps":{"alert":"This is a message for testing APNs","badge":123,"sound":"default"}}
```

These header files should be referenced
```
using System;
using System.IO;
using System.Net.Security;
using System.Net.Sockets;
using System.Security.Authentication;
using System.Security.Cryptography.X509Certificates;
using System.Text;
using System.Threading;
```

- Reading the p12 file which is downloaded from [Apple Developer Website](https://developer.apple.com), the varieble certFilePath is your p12 certificate whole path and the varieble certPwd is the passphrase of certificate, code snippet below
```
X509Certificate2 cert = new X509Certificate2(certFilePath, certPwd);
X509CertificateCollection certificate = new X509CertificateCollection();
certificate.Add(cert);
```

- Then, create an instance of SslStream and handshake before passing host address and port, code snippet below
```
//For distribution mode, the host is gateway.push.apple.com    
//For development mode, the host is gateway.sandbox.push.apple.com
TcpClient client = new TcpClient("gateway.push.apple.com", 2195);

SslStream sslStream = new SslStream(client.GetStream(), false, new RemoteCertificateValidationCallback(ServerCertificateValidationCallback), null);

//The method AuthenticateAsClient() may cause an exception, so we need to try..catch.. it
try
{
    //Reference of SslStream 
    //https://msdn.microsoft.com/en-us/library/system.net.security.sslstream(v=vs.110).aspx?cs-save-lang=1&cs-lang=csharp#code-snippet-2

    sslStream.AuthenticateAsClient(_host, certificate, SslProtocols.Default, false);
}
catch (Exception e)
{
    Console.WriteLine("Exception Message: {0} ", e.Message);
    sslStream.Close();
}

//Obviously , this is a method of callback when handshaking
bool ServerCertificateValidationCallback(object sender, X509Certificate certificate, X509Chain chain, SslPolicyErrors sslPolicyErrors)
{
    if (sslPolicyErrors == SslPolicyErrors.None)
    {
        Console.WriteLine("Specified Certificate is accepted.");
        return true;
    }
    Console.WriteLine("Certificate error : {0} ", sslPolicyErrors);
    return false;
}
```

- Push a remote notification before consisting of the Payload string
```
//This is definition of PushNotificationPayload of struct 
public struct PushNotificationPayload
{
    public string deviceToken;
    public string message;
    public string sound;
    public int badge;

    public string PushPayload()
    {
        return "{\"aps\":{\"alert\":\"" + message + "\",\"badge\":" + badge + ",\"sound\":\"" + sound + "\"}}";
    }
}

//We gave values to it to consisting of the payload content
PushNotificationPayload payload = new PushNotificationPayload();
payload.deviceToken = "dc67b56c eb5dd9f9 782c37fd cfdcca87 3b7bc77c 3b090ac4 c538e007 a2f23a24";
payload.badge = 56789;
payload.sound = "default";
payload.message = "This message was pushed by C# platform.";

//And then, calling Push() method to invoke it
public void Push(PushNotificationPayload payload)
{
    string payloadStr = payload.PushPayload();
    string deviceToken = payload.deviceToken;

    MemoryStream memoryStream = new MemoryStream();
    BinaryWriter writer = new BinaryWriter(memoryStream);

    writer.Write((byte)0); //The command
    writer.Write((byte)0); //The first byte of deviceId length (Big-endian first byte)
    writer.Write((byte)32); //The deviceId length (Big-endian second type)

    //Method of DataWithDeviceToken() , see source code in this repo.
    byte[] deviceTokenBytes = DataWithDeviceToken(deviceToken.ToUpper());
    writer.Write(deviceTokenBytes);

    writer.Write((byte)0); //The first byte of payload length (Big-endian first byte)
    writer.Write((byte)payloadStr.Length); //payload length (Big-endian second byte)

    byte[] bytes = Encoding.UTF8.GetBytes(payloadStr);
    writer.Write(bytes);
    writer.Flush();

    _sslStream.Write(memoryStream.ToArray());
    _sslStream.Flush();

    Thread.Sleep(3000);

    //Method of ReadMessage() , see source code in this repo.
    string result = ReadMessage(_sslStream);
    Console.WriteLine("server said: " + result);

    _sslStream.Close();
}
```



<br/>
<br/>
<br/>
<br/>

Secondly, let me introduce the client side written by C/Objective-C.
See the flow briefly firstly.
- 1.Using `socket()` to connect with Apple Server
- 2.Setting certificate through `SSLSetCertificate()` function within a passphrase
- 3.`SSLHandshake()` with Apple Push Notification server
- 4.`SSLWrite()` for writing data bytes to APNs before splicing a whole string of payload


- Here is the simplest `payload` of a remote notification
```
{"aps":{"alert":"This is a message for testing APNs","badge":123,"sound":"default"}}
```

These header files should be referenced
```
#include <Security/SecureTransport.h>
#include <unistd.h>
#include <netdb.h>
#include <fcntl.h>
```

### Step 1
```
bool connectSocket()
{
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock < 0) {
        printf("Socket creation failed!");
        return false;
    }

    struct sockaddr_in addr;
    memset(&addr, 0, sizeof(struct sockaddr_in));
    struct hostent *entr = gethostbyname(_host);
    if (!entr) {
        printf("Got socket host failed! \n");
        return false;
    }
    struct in_addr host;
    memcpy(&host, entr->h_addr, sizeof(struct in_addr));
    addr.sin_addr = host;
    addr.sin_port = htons((u_short)_port);
    addr.sin_family = AF_INET;
    int conn = connect(sock, (struct sockaddr *)&addr, sizeof(struct sockaddr_in));
    if (conn < 0) {
        printf("Connected to APNs failed! \n");
        return false;
    }
    int cntl = fcntl(sock, F_SETFL, O_NONBLOCK);
    if (cntl < 0) {
        printf("fcntl() function creation failed! \n");
        return false;
    }
    int set = 1, sopt = setsockopt(sock, SOL_SOCKET, SO_NOSIGPIPE, (void *)&set, sizeof(int));
    if (sopt < 0) {
        printf("setsockopt() function set failed! \n");
        return false;
    }
    _socket = sock;
    return true;
}
```

### Step 2
Setting parameters, passphrase and certificate to `SSLContextRef` 
```
bool connectSSLWithCertificate(NSString * certificateFilePath, NSString * certificatePasswords)
{
    SSLContextRef context = SSLCreateContext(NULL, kSSLClientSide, kSSLStreamType);
    if (!context) {
        printf("SSLContextRef creation failed! \n");
        return false;
    }
    OSStatus setio = SSLSetIOFuncs(context, VZSSLRead, VZSSLWrite);
    if (setio != errSecSuccess) {
        printf("OSStatus set failed! \n");
        return false;
    }
    OSStatus setconn = SSLSetConnection(context, (SSLConnectionRef)(unsigned long)_socket);
    if (setconn != errSecSuccess) {
        printf("SSLSetConnection() function failed! \n");
        return false;
    }
    OSStatus setpeer = SSLSetPeerDomainName(context, _host, strlen(_host));
    if (setpeer != errSecSuccess) {
        printf("SSLSetPeerDomainName() function failed! \n");
        return false;
    }

    id certificate = importPKCS12Data(certificateFilePath, certificatePasswords);
    OSStatus setcert = SSLSetCertificate(context, (__bridge CFArrayRef)@[certificate]);
    if (setcert != errSecSuccess) {
        printf("SSLSetCertificate() function failed! \n");
        return false;
    }
    _context = context;
    return true;
}
```

### Step 3
Handshaking
```
bool handshakeSSL()
{
    OSStatus status = errSSLWouldBlock;
    for (int i = 0; i < NWSSL_HANDSHAKE_TRY_COUNT && status == errSSLWouldBlock; i++) {
        status = SSLHandshake(_context);
    }

    bool result = false;
    switch (status) {
        case errSecSuccess: {
            printf("SSLHandshake() success! \n");
            result = true;
        }
        break;
        case errSSLWouldBlock:
        case errSecIO:
        case errSecAuthFailed:
        case errSSLUnknownRootCert:
        case errSSLNoRootCert:
        case errSSLCertExpired:
        case errSSLXCertChainInvalid:
        case errSSLClientCertRequested:
        case errSSLServerAuthCompleted:
        case errSSLPeerCertExpired:
        case errSSLPeerCertRevoked:
        case errSSLPeerCertUnknown:
        case errSecInDarkWake:
        case errSSLClosedAbort: {
            printf("SSLHandshake failed! Failure code = %d \n", status);
            result = false;
        }
        break;
    }
    return result;
}

```

### Step 4
Writting data to APNs server
```
int writePushData(NSData *data, NSUInteger *length)
{
    *length = 0;
    size_t processed = 0;
    OSStatus status = SSLWrite(_context, data.bytes, data.length, &processed);
    *length = processed;
    if (status == errSecSuccess) {
        NSLog(@"%@", securityErrorMessageString(status));
        return errSecSuccess;
    }
    return status;
}
```











