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

