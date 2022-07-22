---
title: "高效的序列化、反序列化工具——ProtoBuf"
date: 2022-07-21T23:01:45+08:00
tags: ['c', 'c++', 'protobuf', '序列化', '反序列化']
categories: ['c&c++']
draft: true
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
- package 声明
.proto 文件以一个 package 声明开始，这个声明是为了防止不同项目之间的命名冲突，对应到 c++中，这个 .proto 文件生成的类将被放置在一个与package名相同的命名空间中。
- 字段类型

https://blog.csdn.net/K346K346/article/details/51754431

### 2. 编译proto文件



### 3. 使用生成的代码来读写消息

