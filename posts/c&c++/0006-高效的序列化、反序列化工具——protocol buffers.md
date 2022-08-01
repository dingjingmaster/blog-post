---
title: "高效的序列化、反序列化工具——ProtoBuf"
date: 2022-07-21T23:01:45+08:00
tags: ['c', 'c++', 'protobuf', '序列化', '反序列化']
categories: ['c&c++']
draft: false
---

## ProtoBuf 简介

ProtoBuf(Protocol Buffers) 是 Google [用于实现`序列化`与`反序列化`的开源项目](https://github.com/protocolbuffers/protobuf#protocol-compiler-installation)，支持多语言、跨平台、可扩展的用于结构化数据的解决方案。

<br/>

目前常见的序列化、反序列化方法包括但不限于以下几种：
- JSON
- XML
- ProtoBuf
- Boost Serialization

## ProtoBuf 数据结构

|proto文件消息类型|C++ 类型|说明|
|---|---|---|
|double|double|双精度浮点型|
|float|float|单精度浮点型|
|int32|int32|使用可变长编码方式，负数时不够高效，应该使用sint32|
|int64|int64|同上|
|uint32|uint32|使用可变长编码方式|
|uint64|uint64|同上|
|sint32|int32|使用可变长编码方式，有符号的整型值，负数编码时比通常的int32高效|
|sint64|sint64|同上|
|fixed32|uint32|总是4个字节，如果数值总是比2^28大的话，这个类型会比uint32高效|
|fixed64|uint64|总是8个字节，如果数值总是比2^56大的话，这个类型会比uint64高效|
|sfixed32|int32|总是4个字节|
|sfixed64|int64|总是8个字节|
|bool|bool||
|string|string|一个字符串必须是utf-8编码或者7-bit的ascii编码的文本|
|bytes|string|可能包含任意顺序的字节数据|

## `.proto` 文件
|关键字|说明|
|---|---|
|syntax|指定`proto`语言版本|
|option|修改配置选项|
|service|声明一个服务|
|rpc|声明一个方法|
|resturns|方法的返回值|
|message|定义一个消息类型|
|repeated|数组|
|stream|用流来交互|

### 一些例子

#### 指定一个版本
```
syntax = "proto3"
```

#### 定义一个服务和方法
```
service TestService 
{
    rpc testMethod(Request) returns (Result) {}
}

message Request
{
}

message Result
{
}
```

## ProtoBuf使用一般步骤

### 1. 定义proto文件

proto文件中就是定义了我们需要存储或传输的数据结构/传输协议

<br/>
proto文件的定义主要分为两部分：

1. 为每一个需要序列化的数据结构添加一个消息(message)。
2. 为消息(message)中的每一个字段(field)指定一个名字、类型和修饰符以及唯一标识(tag)。

<br/>
其中每一个消息对应到C++就是一个类，嵌套消息对应的就是嵌套类。

<br/>
另外，一个proto文件中可以定义多个消息，就像一个头文件中可以定义多个类一样。

```protobuf
// [START declaration]
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";
// [END declaration]

// [START java_declaration]
option java_multiple_files = true;
option java_package = "com.example.tutorial.protos";
option java_outer_classname = "AddressBookProtos";
// [END java_declaration]

// [START csharp_declaration]
option csharp_namespace = "Google.Protobuf.Examples.AddressBook";
// [END csharp_declaration]

// [START go_declaration]
option go_package = "github.com/protocolbuffers/protobuf/examples/go/tutorialpb";
// [END go_declaration]

// [START messages]
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}
// [END messages]

```
- package 声明：`.proto` 文件以一个 `package` 声明开始，这个声明是为了防止不同项目之间的命名冲突，对应到 c++ 中，这个 .proto 文件生成的类将被放置在一个与package名相同的命名空间中。
- 字段类型：定义 message 时候，一个 message 就是某些类型字段的集合。具体支持的字段类型见：[字段类型](/2022/07/0006-%E9%AB%98%E6%95%88%E7%9A%84%E5%BA%8F%E5%88%97%E5%8C%96%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E5%B7%A5%E5%85%B7protocol-buffers/#protobuf-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
- 修饰符：每个字段都必须用以下之一的修饰符来修饰：
|修饰符|含义|
|---|---|
|required|必须提供字段值，否则对应的消息会被认为是"`未初始化`"的。<br/>注意：`解析"未初始化"的消息会导致失败`。|
|optional|字段值指定与否都可以，它是每个字段默认值。<br/>调用时候没有指定message的字段值，会自动使用默认值：`string`默认值是`空字符串`；`int`默认值是`0`|
|repeated|字段会重复`N`次(`N`可以是`0`)，重复值的顺序会被保存在ProtoBuf中<br/>注意：`可以把重复字段视为数组`。|
- 唯一编号：每个消息中的每个字段都有唯一的编号，字段后边的`=1`、`=2`等。这些字段编号用于标识消息二进制中的字段，并且在使用消息类型后不应该再更改。注意：`1 ~ 15` 中的字段需要一个字节进行编码，包括字段编号和字段类型。`16 ~ 2047` 中的字段编号需要两个字节。所以保留 1 - 15 作为非常频繁出现的消息元素，也要注意为将来可能频繁出现的消息元素预留位置。
> 可以指定的最小字段编号为：`1`，最大字段编号：$2^{29}-1$ 或 `536 870 911`。
> <br/>也不能使用数字`19 000` 到 `19 999`，他们是 protobuf 保留的。


### 2. 编译proto文件

```shell
protoc --cpp_out=/data/code/demo/protobuf/c++ -I/data/code/demo/protobuf/c++ xxx.proto
```
其中：
- `--cpp_out=<dir>` 表示生成代码输出到的指定文件夹
- `-I<dir>` 表示在哪个文件夹下寻找 xxx.proto 文件
- `xxx.proto` 就是我们写好的 `.proto` 文件

[完整例子](https://github.com/dingjingmaster/demo/tree/master/protobuf)

> 注意：如果编译上述例子出错，则可以重新编译安装 protobuf，几乎没有依赖，克隆源码就可编译安装。
> <br/> 亲测我的 `archlinux` 使用包管理器安装的 protobuf 是不能用的。

编译 protobuf 源码具体步骤如下：
1. 克隆源码：[https://github.com/protocolbuffers/protobuf.git](https://github.com/protocolbuffers/protobuf.git)
2. 进入源码目录执行安装：`./autogen.sh && ./configure --prefix=/usr && make -j8 && sudo make install`


### 3. 使用生成的代码来读写消息

```c
// [1] 验证版本
GOOGLE_PROTOBUF_VERIFY_VERSION;         

// [2] 根据 protobuf 中 message 生成的对象
test1::Response p1;
p1.set_data("data");
p1.set_status(1);

cout << "=================================\n";
cout << "p1.data: " << p1.data() << "\n"
     << "p1.status: " << p1.status() << "\n";

// [3] 执行序列化
// 注意：序列化之后可以用 p1.DebugString();  来查看序列化字符串
cout << "serialize string:" << p1.SerializeAsString() << "\n";
cout << "=================================\n";

// [4] 反序列化
// 反序列化之前也可以使用 DebugString() 来查看要反需列化的字符串是否正确
// 反序列化 bool ParseFromString(const string& data);

// [5] 释放 protobuf 相关内存
google::protobuf::ShutdownProtobufLibrary();

```

> 序列化与反序列化过程中尽量保证使用 `char*` 或 `std::string` 来接收数据。
> <br/>亲测QString参与会导致反序列化失败。

[完整例子](https://github.com/dingjingmaster/demo/tree/master/protobuf)
