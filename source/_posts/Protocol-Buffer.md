---
title: Protocol Buffer
date: 2017-10-18 22:52:27
tags: Protocol Buffers, protobuff
categories: iOS
---

## 目录
- 1.<a href="#introduction">介绍</a>
- 2.<a href="#downloadAndInstallation">下载与安装</a>
- 3.<a href="#examplePython">实例说明（Python代码）</a>
- 4.<a href="#exampleOC">实例说明（Objective-C代码）</a>
- 5.<a href="#explanationProtoFile">文件.proto的解释</a>

<br/>

## <span id="introduction"></span>1.介绍
<a href="https://developers.google.com/protocol-buffers/docs/overview" target="">Protocol Buffer</a>，简单来说，就是一种数据交换格式，就像`JSON`和`XML`作用一样，只不过<a href="https://developers.google.com/protocol-buffers/docs/overview" target="">Protocol Buffer</a>是Google开源的一套二进制流网络传输协议，它独立于语言，独立于平台；而且它的性能、速度等非常优越于`JSON`和`XML`。google 提供了多种语言的实现：objective-c, swift,java、c#、c++、Go 和Python等，每一种实现都包含了相应语言的编译器以及库文件。由于它是一种二进制的格式，比使用 xml 进行数据交换快许多。可以把它用于分布式应用之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域。

对于XML来说，`Protocol Buffers`有太多优点了，尤其是针对序列化结构数据。优点：
- 更简单
- 小到3-10倍
- 快到20-100倍
- 相当少的歧义
- 文档型协议
- T-L-V的数据存储方式 Tag-Length-Value

## 注意
`protobuf`目前有两个版本`proto2`和`proto3`。这两个版本的语法并不是完全兼容的，所以为了避免使用时的麻烦，请仔细阅读<a href="https://developers.google.com/protocol-buffers/docs/proto" target="_blank">proto2</a>和<a href="https://developers.google.com/protocol-buffers/docs/proto3" target="_blank">proto3</a>的语法。

<br/>

## <span id="downloadAndInstallation"></span>2.下载与安装
我的环境：macOS Sierra Version 10.12.6
- 1.<a href="https://github.com/google/protobuf/releases/tag/v3.4.1" target="_blank">下载</a>最新版本的ProtoBuff (我下载的是v3.4.1)。下载完后，解压压缩包。`注意`：解压后里面的`objectivec目录`是给`iOS`和`macOS`专用
- 2.然后依次键入以下命令进行安装，以下的每个命令都可能或占用几分钟时间，请耐心等待下
```
$> cd protobuf-3.4.1/
$> ./configure
$> make
$> make check
$> make install

// 当输入此步骤时，如果正常输出版本号信息，就表示安装正确了
$> protoc --version
```

<br/>

## Python的使用命令
<a href="https://developers.google.com/protocol-buffers/docs/pythontutorial" target="_blank">Python教程</a>
编译一个`.proto`文件，命令如下：
```
//$SRC_DIR 为源文件目录
//$DST_DIR 为目标文件目录
protoc -I=$SRC_DIR --python_out=$DST_DIR $SRC_DIR/addressbook.proto
```
然后，你就会看到后缀为：`_pb2.py`的文件在你指定的目录下。

<br/>

## Objective-C的使用命令
<a href="https://developers.google.com/protocol-buffers/docs/reference/objective-c-generated" target="_blank">Objective-C教程</a>
编译一个`.proto`文件，命令如下：
```
//$SRC_DIR 为源文件目录
//$DST_DIR 为目标文件目录
protoc --proto_path=$SRC_DIR --objc_out=$DST_DIR $SRC_DIR/addressbook.proto
```
然后，你就会看到后缀为：`.pbobjc.h`和`.pbobjc.m`的文件在你指定的目录下。


<br/>
<br/>

==========================================================================================
## <span id="examplePython"></span>3.实例说明（Python代码）
我们先展示一段服务器和客户端通信的代码，使用TCP/IP协议。通常的代码，简化如下：
**服务器端**
```
#  coding:utf-8

from socket import *

_HOST = '127.0.0.1'
_PORT = 21567
_BUFSIZE = 4096
_ADDR = (_HOST, _PORT)

# 创建socket -> 地址绑定 -> 监听客户端
_tcpSerSock = socket(AF_INET, SOCK_STREAM)
_tcpSerSock.bind(_ADDR)
_tcpSerSock.listen(5)

# 这里就假设只连接上一个客户端
print "Waiting for connection..."
_tcpCliSock, _cliAddr = _tcpSerSock.accept()
print "connected from :", _cliAddr

# 然后向该客户端写入数据
_tcpCliSock.send('hello, client')

_cliData = _tcpCliSock.recv(_BUFSIZE)
print "服务器端收到数据为: ", _cliData

# 把客户端连接断开
_tcpCliSock.close()
print "Coerce closed client connection."

# 服务器端关闭连接
_tcpSerSock.close()
print "Server closed."

```

**客户端**
```
#  coding:utf-8

from socket import *

_HOST = '127.0.0.1'
_PORT = 21567
_BUFSIZE = 4096
_ADDR = (_HOST, _PORT)

# 创建客户端Socket，并连接上
_tcpCliSock = socket(AF_INET, SOCK_STREAM)
_tcpCliSock.connect(_ADDR)

# 假设收到服务器端的消息后，就回复一句，然后关闭连接
_data = _tcpCliSock.recv(_BUFSIZE)
print "客户端收到数据为：", _data

_tcpCliSock.send("Hey, server, I received your message.")

_tcpCliSock.close()
print "client connection closed."
```
这段代码很简单，就是客户端和服务器连接上后，客户端会收到一条`服务器端`发送的消息，服务器也收到`客户端`发送的一条消息。
我们平时假如要使用socket发送消息时，一般就是json或者xml字符串的形式发送，两端都需要文档来解释`json`或者`xml`字符串里的每个字段代表的意思，而且两端都需要`包装json字符串`和`解析json字符串`，xml也是一样。

那么此时，我们有一个更好的选择，就是使用Protocol Buffer，它的性能更快，更好，更简单，而且文档即协议，（我就不再赘述`protobuff`的好处了，因为该篇文章上面已经解释了），不需要额外的文档来说明传递的消息的字段，因为在`.proto`文件里就可以写上注释，表示相应的意思。

** 使用Protocol Buffer来交互 **
### 1). 准备proto文件
<a href="https://developers.google.com/protocol-buffers/docs/pythontutorial" target="_blank">以下数据由protocol buffers官方网站提供</a>
首先我们需要创建一个`.proto`文件，内容如下
```
syntax = "proto2";

package tutorial;

message Person {
    required int32 id = 1;
    required string name = 2;
    optional string email = 3;

    enum PhoneType {
        MOBILE = 0;
        HOME = 1;
        WORK = 2;
    }

    message PhoneNumber {
        required string number = 1;
        optional PhoneType type = 2 [default = HOME];
    }

    repeated PhoneNumber phones = 4;
}

message AddressBook {
    repeated Person people = 1;
}
```
然后运行命令如下:
```
protoc -I=./ --python_out=./ addressbook.proto
```
就会生成后缀为`_pb2.py`的文件

**服务器端**
```
#  coding:utf-8

from socket import *
import addressbook_pb2


# 1.创建AddressBook对象，并赋值
_address_book = addressbook_pb2.Person()
_address_book.id = 123456987
_address_book.name = "Victor Zhang"
_address_book.email = "victorzhangq@gmail.com"
_phone_number = _address_book.PhoneNumber()
_phone_number.number = "18711112222"
_phone_number.type = addressbook_pb2.Person.MOBILE

# 序列化数据
_binaryStr = _address_book.SerializeToString()


_HOST = '127.0.0.1'
_PORT = 21567
_BUFSIZE = 4096
_ADDR = (_HOST, _PORT)

# 创建socket -> 地址绑定 -> 监听客户端
_tcpSerSock = socket(AF_INET, SOCK_STREAM)
_tcpSerSock.bind(_ADDR)
_tcpSerSock.listen(5)

# 这里就假设只连接上一个客户端
print "Waiting for connection..."
_tcpCliSock, _cliAddr = _tcpSerSock.accept()
print "connected from :", _cliAddr

# 然后向该客户端写入数据
_tcpCliSock.send(_binaryStr)
print "已发送", len(_binaryStr)

_cliData = _tcpCliSock.recv(_BUFSIZE)
print "服务器端收到数据为: ", _cliData

# 把客户端连接断开
_tcpCliSock.close()
print "Coerce closed client connection."

# 服务器端关闭连接
_tcpSerSock.close()
print "Server closed."

```

**客户端代码**
```
#  coding:utf-8

from socket import *
import addressbook_pb2

_address_book = addressbook_pb2.Person()

_HOST = '127.0.0.1'
_PORT = 21567
_BUFSIZE = 4096
_ADDR = (_HOST, _PORT)

# 创建客户端Socket，并连接上
_tcpCliSock = socket(AF_INET, SOCK_STREAM)
_tcpCliSock.connect(_ADDR)

# 假设收到服务器端的消息后，就回复一句，然后关闭连接
_data = _tcpCliSock.recv(_BUFSIZE)
# 反序列化消息
_address_book.ParseFromString(_data)
print "客户端收到数据为：", _address_book

_tcpCliSock.send("Hey, server, I received your message.")

_tcpCliSock.close()
print "client connection closed."
```

测试时，记得一定要先启动服务器端`python server.py`，然后再启动客户端`python client.py`，启动完客户端后你就能看到客户端收到服务器发过来的`protocol buffer`的数据了。

<br/>
<br/>

==========================================================================================
## <span id="exampleOC"></span>4.实例说明（Objective-C代码）
我这里写了`iOS`和`macOS`分别两个项目作为实例代码，以`Objective-C`代码编写的，socket服务器端和客户端交互，<a href="https://github.com/VictorZhang2014/ProtoBuf_Demo" target="_blank">下载地址</a>

**1).先来对Xcode配置一下**
- 新建一个项目，在项目目录下，建一个`ProtoBuf`目录
- 在<a href="#downloadAndInstallation">第二步</a>里找到`objectivec`目录，并且把这整个目录里所有的文件都拷贝到项目的`ProtoBuf`下
    - 然后按照下图，依次配置
![protocbuf_step1](/img/iOS/protobuf/protocbuf_step0.png)
![protocbuf_step1](/img/iOS/protobuf/protocbuf_step1.png)
![protocbuf_step1](/img/iOS/protobuf/protocbuf_step2.png)
![protocbuf_step1](/img/iOS/protobuf/protocbuf_step3.png)
![protocbuf_step1](/img/iOS/protobuf/protocbuf_step4.png)


**1).Person.proto文件内容**
先准备的`.proto`的文件，如下：
```
syntax = "proto3";

message Person{
	string name = 1;
	int32 age = 2;
    float height = 3;

	enum DeviceType{ 
		IOS = 0;
		Android = 1;
		WP = 2;
	}

	DeviceType deviceType = 4;

	message Result{
		string url = 1;
		string title = 2;
	}
	repeated Result results = 5;

	repeated Animal animals = 6;
}

message Animal{
	double price  = 2;
	string name = 3;
}
```

**服务器端**
```
#import <Foundation/Foundation.h>
#import <sys/socket.h>
#import <netdb.h>
#import "Person.pbobjc.h"

NSData *  serialize();
void deserialize();

int main(int argc, const char * argv[]) {
    @autoreleasepool {

        //1.创建socket
        int socketFD = socket(AF_INET, SOCK_STREAM, 0);
        
        struct sockaddr_in addr;
        memset(&addr, 0, sizeof(addr));
        addr.sin_len = sizeof(addr);
        addr.sin_family = AF_INET;
        addr.sin_port = htons(6789);
        addr.sin_addr.s_addr = INADDR_ANY; //指定监听的ip，指定为INADDR_ANY时，表示监听所有的ip
        
        //2.绑定并监听
        int error = -1;
        error = bind(socketFD, (const struct sockaddr *)&addr, sizeof(addr));
        error = listen(socketFD, 5);
        printf("接收客户端连接中。。。\n");
        
        //3.接收客户端连接
        struct sockaddr_in peerAddr;
        socklen_t addrLen = sizeof(peerAddr);
        int clientSocketFD = accept(socketFD, (struct sockaddr *)&peerAddr, &addrLen);
        
        //4.接收数据
        void * buf = malloc(1024);
        size_t len = sizeof(buf);
        read(clientSocketFD, buf, 1024);
        NSData *recData = [NSData dataWithBytes:buf length:1024];
        deserialize(recData);
        
        //5.发送数据到客户端
        NSData *pendingData = serialize();
        ssize_t re = write(clientSocketFD, [pendingData bytes], pendingData.length);
        if (re == pendingData.length) {
            NSLog(@"发送成功！");
        }
        
        close(clientSocketFD);
        close(socketFD);
    }
    return 0;
}

//序列化数据
NSData * serialize()
{
    //1.先初始化一些值吧
    Person *person = [[Person alloc] init];
    person.name = @"Victor张 - 服务器";
    person.age = 24;
    person.height = 185;
    person.deviceType = Person_DeviceType_Android;
    
    Person_Result *p_result = [[Person_Result alloc] init];
    p_result.title = @"我的博客 - 服务器";
    p_result.URL = @"http://www.googleplus.party/";
    [person.resultsArray addObject:p_result];
    
    Person_Result *p_result1 = [[Person_Result alloc] init];
    p_result1.title = @"我的Facebook - 服务器";
    p_result1.URL = @"https://www.facebook.com/victor.john.92167789?ref=bookmarks";
    [person.resultsArray addObject:p_result1];
    
    Animal *animal = [[Animal alloc] init];
    animal.price = 109;
    animal.name = @"Ketty - 服务器";
    [person.animalsArray addObject:animal];
    
    //序列化后的二进制数据
    NSData *data = [person delimitedData];
    
    return data;
}

//反序列化数据
void deserialize(NSData *data)
{
    //将二进制数据反序列化成对象
    NSError *error = nil;
    GPBCodedInputStream *inputStream = [GPBCodedInputStream streamWithData:data];
    Person *de_person = [Person parseDelimitedFromCodedInputStream:inputStream extensionRegistry:nil error:&error];
    NSLog(@"接受到数据，并反序列化：%@", de_person);
}
```

**客户端**
```
#import "ViewController.h"
#import <sys/socket.h>
#import <netdb.h>
#import "Person.pbobjc.h"

@interface ViewController ()

@property (nonatomic, assign) int socketFD;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    //1.先连接上socket
    self.socketFD = socket(AF_INET, SOCK_STREAM, 0);
    
    struct hostent * remoteHostEnt = gethostbyname("127.0.0.1");
    struct in_addr * remoteInAddr = (struct in_addr *)remoteHostEnt->h_addr_list[0];
    struct sockaddr_in socketParameters;
    socketParameters.sin_family = AF_INET;
    socketParameters.sin_addr = *remoteInAddr;
    socketParameters.sin_port = htons(6789);
    
    int ret = connect(self.socketFD, (struct sockaddr *) &socketParameters, sizeof(socketParameters));
    if (-1 == ret) {
        close(self.socketFD);
        NSLog(@"连接socket失败！");
        return;
    }
    NSLog(@"已经连接上服务器！");
    
    [self serialize];
    [self deserialize];
}

//序列化
- (void)serialize
{
    //1.先初始化一些值吧
    Person *person = [[Person alloc] init];
    person.name = @"Victor张";
    person.age = 25;
    person.height = 175;
    person.deviceType = Person_DeviceType_Ios;
    
    Person_Result *p_result = [[Person_Result alloc] init];
    p_result.title = @"我的博客";
    p_result.URL = @"http://www.googleplus.party/";
    [person.resultsArray addObject:p_result];
    
    Person_Result *p_result1 = [[Person_Result alloc] init];
    p_result1.title = @"我的Facebook";
    p_result1.URL = @"https://www.facebook.com/victor.john.92167789?ref=bookmarks";
    [person.resultsArray addObject:p_result1];
    
    Animal *animal = [[Animal alloc] init];
    animal.price = 109;
    animal.name = @"Ketty";
    [person.animalsArray addObject:animal];
    
    //序列化后的二进制数据
    NSData *data = [person delimitedData];
    
    //向socket通道写入二进制数据
    ssize_t re = write(self.socketFD, [data bytes], data.length);
    if (re == data.length) {
        NSLog(@"发送成功！");
    }
}

//反序列化
- (void)deserialize
{
    //从socket通道中接收数据
    void * buf = malloc(1024);
    size_t len = sizeof(buf);
    read(self.socketFD, buf, 1024);
    NSData *data = [NSData dataWithBytes:buf length:1024];
    
    
    //将二进制数据反序列化成对象
    NSError *error = nil;
    GPBCodedInputStream *inputStream = [GPBCodedInputStream streamWithData:data];
    Person *de_person = [Person parseDelimitedFromCodedInputStream:inputStream extensionRegistry:nil error:&error];
    NSLog(@"%@", de_person);
    
    //关闭socket
    close(self.socketFD);
}
@end
```
这些代码就比较简单了，就不作解释了，在python的那一块已经解释过了。


<br/>
<br/>

==========================================================================================
## <span id="explanationProtoFile"></span>5.文件.proto的解释
## Proto2
<a href="https://developers.google.com/protocol-buffers/docs/proto" target="_blank">Proto2官方文档</a>

**实例文件**
```
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 result_per_page = 3;
}
```

**分配标签**
从上面的代码，可以看出，每个字段后面都有一个数字，而是是递增的；官方说：这是个`唯一数字标签`，数字在`1到15`范围内，会占用一个字节去编码，如果是`16到2047`范围内，则是占用两个字节去编码。

**指定每个字段的规则**
- `required`，一个结构良好的message，必须有`required`修饰
- `optional`，一个结构良好的message，可以是`没有optional`或者`一个optional`修饰
- `repeated`，意思是，对修饰的字段进行多次重复，也就是数组，动态数组。
  - 因为历史原因，官方建议在使用`repeated`修饰时，一定要类似如下这样写：
  ```
    repeated int32 samples = 4 [packed=true];
  ```
  至于`packed`的意思，你可以在<a href="https://developers.google.com/protocol-buffers/docs/encoding#structure" target="_blank">这里</a>了解到

**添加多个message类型**
例如如下代码， 有时候你想在`.proto`文件里，添加多个`messge类型，可能由于业务需要，这是完全可以的
```
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```

**添加注释**
`protobuf`是面向协议文档的，所以，你完全可以在`.proto`里写上相应的注释，放在以前，如果json格式，你还需要专门用一个文档来记录，json的键值对各个意思，protobuf就不用这么麻烦，直接写到`.proto`文件里即可。
注释的方式：`//...`和`/* ... */`

**reserved使用**
当你在调整业务时，可能某些字段用不到了，记住，千万不要直接移除，然后还把他的tag number给别的字段使用，这会导致非常致命的错误。
解决办法：以下展示了，那些tag number不在使用了，哪些字段被弃用了
```
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

**标量值类型**
![Scalar Type](/img/iOS/protobuf/protobuf_scalar_types1.png)
![Scalar Type](/img/iOS/protobuf/protobuf_scalar_types2.jpeg)

**设置默认值**
为`optional`修饰的字段做默认值，如果你不写默认值，则解析时自动填上
```
optional int32 result_per_page = 3 [default = 10];
```

**枚举**
例如：这样的枚举
```
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 result_per_page = 3 [default = 10];
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  optional Corpus corpus = 4 [default = UNIVERSAL];
}
```
枚举里的tag number，也就是唯一数字不能重复，如果数字重复，则protobuf认为你是想做一个别名，如果想做一个别名，则需要按照以下方式写
```
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
```
假如，你没有写`option allow_alias = true;` 则会引起编译错误。
- 枚举值的范围32位整型数字
- 如果你设置了负数，则是非常影响效率的

**message类型嵌套**
```
message SearchResponse {
  repeated Result result = 1;
}

message Result {
  required string url = 1;
  optional string title = 2;
  repeated string snippets = 3;
}
```

**导入message定义文件**
如果你在其他的`.proto`文件里使用到了另外一个类里的`.proto`里的message，那么你可以通过`import`导入，例如
```
import "myproject/other_protos.proto";
```

**使用proto3的message类型**
- 在proto2文件里，可以使用proto3的message，反过来也一样。但是proto2的枚举语法不能在proto3里用


**嵌套类型**
你可以在一个message里嵌套另个一个message，如
```
message SearchResponse {
  message Result {
    required string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
  }
  repeated Result result = 1;
}
```
只要你喜欢，多层嵌套，也是可以的

**更新一个message类型**
更新一个已经存在的message类型，也很简单，只需要注意以下几点
- 不要改变任何一个已经存在的field的数字tag
- 一个新添加的field应该是`optional`或者`repeated`，而且还需要设置一个默认值
- 不需要的field可以被移除，只要这个被移除的field的tag number你不在使用即可
- 如int32, uint32, int64, uint64, and bool直接可以相互的修改类型声明，
还有更多，看文档吧

**extensions**
扩展message
```
message Foo {
  // ...
  //Foo的message的值范围是在100到199之间
  extensions 100 to 199;
  optional int32 age = 120;
}
```
扩展message，添加更多的field在另一个文件
```
extend Foo {
  optional string name = 130;
}
```

**oneof**


**map**
map可以理解为字典，你在定义field的时候，如果用到字典类型的话，就用map，例如
```
map<string, Project> projects = 3;
```
- map不支持`repeated`, `optional`, 或者`required`


**options**
- file-level
- message-level
- field-level







## Proto3
<a href="https://developers.google.com/protocol-buffers/docs/proto3" target="_blank">Proto3官方文档</a>

**定义一个message类型**
很简单，如下所示：
```
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

**分配标签**
从上面的代码，可以看出，每个字段后面都有一个数字，而是是递增的；官方说：这是个`唯一数字标签`，数字在`1到15`范围内，会占用一个字节去编码，如果是`16到2047`范围内，则是占用两个字节去编码。


**field规则**
- repeated 跟proto2一样，就相当于数组一样，但不是数组

**添加多个message**
跟proto2一样

**添加注释**
跟proto2一样

**保留field**
跟proto2一样

**默认值**
- string，就是empty string
- bytes，就是empty bytes
- bool，就是false
- 数字类型，就是0
- 枚举，就是枚举的第一个选项值
- message，取决于语言

**导入proto定义**
跟proto2一样

**嵌套message类型**
跟proto2一样

**更新一个message类型**
跟proto2一样

**Any类型**
google还在开发中，暂时不建议使用
```
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

**map**
跟proto2一样

**JSON映射**
proto3支持典型的JSON编码，可以使它更容易的在系统之间分享数据，下面有一张表来表示`protobuf`与`JSON`对应的数据类型

![protobuf3_scalar_types1](/img/iOS/protobuf/protobuf3_scalar_types1.png)
![protobuf3_scalar_types2](/img/iOS/protobuf/protobuf3_scalar_types2.png)
![protobuf3_scalar_types3](/img/iOS/protobuf/protobuf3_scalar_types3.png)

