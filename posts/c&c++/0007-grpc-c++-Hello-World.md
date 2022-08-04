---
title: "gRPC C++ 例子"
date: 2022-08-01T22:31:15+08:00
tags: ['c++', 'grpc']
categories: ['c\c++']
draft: false
---

## 说明

gRPC 是一款好用的 `RPC` 框架，它使用 `protobuf` 做为接口定义和底层消息交换格式。

使用 gRPC，一个客户端程序可以直接调用另一台机器上服务端程序的方法，这使的你可以很容易创建分布式程序和服务。

与大多数 `RPC` 框架一样，`gRPC` 定义一个service的方法是：指定方法、远程调用参数和返回值。在服务端这边，需要实现定义的接口，并且运行一个grpc server 去处理客户端请求；在客户端这边有一个与服务端相同方法的 stub，通过这个方法获取服务端返回值。

<div align=center><img src='/pic/c&c++/11.svg'/></div>
<center>gRPC调用</center>

## gRPC c++环境搭建

### 环境准备

- cmake
- autoconf libtool pkg-config
- 其它一些编译必须的环境

### 克隆代码

```shell
git clone --recurse-submodules -b v1.46.3 --depth 1 --shallow-submodules https://github.com/grpc/grpc
```

### 下载依赖的源码

```shell
# 在源码目录执行，第一步克隆时候没有下载子项目的执行下面命令可以下载
make -j8
```

### 编译

```shell
# 在源码目录下执行
mkdir build && cd build && cmake .. && make -j8
```
> 编译后安装: `sudo make install`，默认安装到`/usr/local/`下，而不是`/usr`下，所以使用时候要注意。
> <br/>我这有一例子，可供参考：
> [https://github.com/dingjingmaster/demo/tree/master/grpc](https://github.com/dingjingmaster/demo/tree/master/grpc)
> <br/>可以看 `demo1/` 目录下的 `Makefile`


## 与`protocol Buffers`一起使用的例子

定义一个`.proto`文件，gRPC 提供了 proto buffer 编译插件用来产生客户端和服务端代码。使用 gRPC 通常是在客户端调用这些产生的接口并且在服务端实现这些接口。

### 1. `protobuf` 定义

```proto
syntax = "proto3";

service MsgService {
    rpc GetMsg (MsgRequest) returns (MsgResponse) {}
}

message MsgRequest {
    string name = 1;
    int32 num1 = 2;
    double num2 = 3;
}

message MsgResponse {
    string msg = 1;
    int32 num1 = 2;
    double num2 = 3;
}
```
### 2. 生成服务端代码

```shell
# makefile 里部分代码
protoc -I $(cur_dir) --grpc_out=$(cur_dir) --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` msg.proto
```

### 3. 实现服务端

1. 继承生成服务端里的服务端类
2. 实现在`proto`文件里定义的方法
3. 绑定端口并运行

```c++
#include <string>
#include <memory>
#include <iostream>

#include <stdio.h>

#include <grpcpp/grpcpp.h>

#include "msg.pb.h"
#include "msg.grpc.pb.h"

using grpc::Server;
using grpc::Status;
using grpc::ServerBuilder;
using grpc::ServerContext;

class MyMsgServer final : public MsgService::Service
{
    virtual Status GetMsg (ServerContext* ctx, const MsgRequest* req, MsgResponse* rsp) override
    {
        std::string str1 ("Hello ");
        rsp->set_msg (str1 + req->name());
        rsp->set_num1 (32);
        rsp->set_num2 (3.1415);

        return Status::OK;
    }
};

void RunServer ()
{
    std::string serverAddr("0.0.0.0:50000");

    MyMsgServer service;

    // 创建工厂类
    ServerBuilder builder;

    // 监听端口
    builder.AddListeningPort (serverAddr, grpc::InsecureServerCredentials());

    // 注册服务
    builder.RegisterService(&service);

    // 创建和启动一个RPC服务器
    std::unique_ptr<Server> server(builder.BuildAndStart());

    std::cout << "server listening on " << serverAddr << std::endl;

    server->Wait();
}


int main (int argc, char* argv[])
{
    RunServer();

    return 0;
}
```

### 4. 生成客户端代码

```shell
# makefile 里的部分代码
g++ $^ -L/usr/local/lib `pkg-config --with-path=$(pkg_path) --cflags --libs grpc++ protobuf` -lpthread -lgrpc++_reflection -ldl -o $@
```

### 5. 实现客户端调用服务端

```c++
#include <memory>
#include <string>
#include <iostream>

#include <grpcpp/grpcpp.h>

#include "msg.grpc.pb.h"

using grpc::Channel;
using grpc::ClientContext;
using grpc::Status;


class MsgClient
{
public:
    MsgClient (std::shared_ptr<Channel> channel) : stub_(MsgService::NewStub(channel))
    {
    }

    MsgResponse GetMsg (const std::string& user, int num1, double num2)
    {
        MsgRequest request;

        request.set_name(user);
        request.set_num1(num1);
        request.set_num2(num2);

        // 服务器返回端
        MsgResponse reply;

        ClientContext context;

        Status status = stub_->GetMsg(&context, request, &reply);

        if (status.ok()) {
            return reply;
        } else {
            std::cout << status.error_code() << ": " << status.error_message() << std::endl;

            return reply;
        }
    }

private:
    std::unique_ptr<MsgService::Stub> stub_;
};

int main (int argc, char* argv[])
{
    MsgClient z_msg (grpc::CreateChannel("127.0.0.1:50000", grpc::InsecureChannelCredentials()));

    std::string user("world");

    MsgResponse reply = z_msg.GetMsg(user, 234, 3.14);

    std::cout << reply.msg() << std::endl;

    printf ("num1 = %d; num2 = %f\n", reply.num1(), reply.num2());

    return 0;
}
```

## gRPC中一些概念

### `.proto`文件中的写法

- 当客户端发送一个请求到服务端并获取响应时候使用`rpc`定义一个方法。
```
rpc SayHello(HelloRequest) returns (HelloResponse);
```
- 服务器流rpc，客户机向服务器发送一个请求，来读取返回的消息序列，grpc 保证了返回数据的顺序与定义的一致。
```
rpc LostOfReplies(HelloRequest) returns (stream HelloResponse);
```
- 客户端流rpc，客户端发送请求后会一直等待服务端返回结果
```
rpc LostsOfGreetings (stream HelloRequest) returns (HelloResponse);
```
- 双向流RPCs, 双方使用`读-写`流发送消息。这两个流独立操作，因此客户端和服务端可以按照自己喜欢的顺序进行读写。
```
rpc BidiHello (stream HelloRequest) returns (stream HelloResponse);
```


### 同步与异步

同步RPC(synchronous RPC)调用会阻塞，直到服务端那边返回消息。另一方面，网络本身是异步的，在许多场景中，能够在不阻塞当前线程的情况下启动RPC是很有用的。大多数语言中的gRPC编程API有同步和异步两种形式。您可以在每种语言的教程和参考文档中找到更多信息。

### RPC生命周期

本节你将进一步了解 gRPC 客户端调用 gRPC 服务器时候会发生什么。

#### 最简单的RPC

客户端发送**一个请求**并且阻塞等待**一个响应**。

1. 一旦客户端调用了服务器方法，服务器会收到通知，RPC已经被调用，其中包括客户端用于该调用的元数据、方法名称和指定的截止日期(如果使用的话)。
2. 然后服务器可以直接发送回自己的初始元数据(必须在任何响应之前发送)，或者等待客户机的请求消息。首先发生的是特定于应用程序的。
3. 一旦服务器获得了客户机的请求消息，它就会执行创建和填充响应所需的任何工作。然后将响应连同状态详细信息和可选尾随元数据一起返回给客户机。
4. 如果响应状态为OK，那么客户端将获得响应，从而在客户端完成调用。

#### 服务端流式RPC

服务器流RPC类似于一元RPC，不同的是服务器返回一个消息流来响应客户端的请求。在发送完所有消息后，服务器的状态详细信息(状态代码和可选的状态消息)和可选的尾随元数据被发送给客户端。这样就完成了服务器端的处理。一旦客户端获得了服务器的所有消息，它才算执行完成了。

#### 客户端流式RPC

客户端流RPC类似于一元RPC，不同之处是客户端向服务器发送消息流而不是单个消息。服务器响应一条消息(以及它的状态详细信息和可选的尾随元数据)通常但不一定是在它收到客户端的所有消息之后。

#### 双向流RPC

在双向流RPC中，调用方法的客户端和接收客户端元数据、方法名称和截止日期的服务器发起调用。服务器可以选择发送回它的初始元数据，或者等待客户机开始流消息。

客户端和服务器端流处理是特定于应用程序的。由于两个流是独立的，客户端和服务器可以以任何顺序读取和写入消息。例如，服务器可以等待，直到它收到了客户端的所有消息才写入它的消息，或者服务器和客户端可以打“乒乓”——服务器获得一个请求，然后发回一个响应，然后客户端根据响应发送另一个请求，以此类推。

#### 截止时间/超时

gRPC允许客户端指定在RPC因`DEADLINE_EXCEEDED`错误而终止之前，他们愿意等待多长时间来完成RPC。在服务器端，服务器可以查询查看某个特定RPC是否超时，或者还剩多少时间来完成RPC。

指定截止时间或超时是特定于语言的:一些语言api根据超时(时间持续时间)工作，而一些语言api根据截止时间(一个固定的时间点)工作，可能有也可能没有默认的截止时间。

#### RPC终止

在gRPC中，客户端和服务器端对调用的成功与否进行独立的、本地的判断，其结论可能不匹配。这意味着，例如，您可能有一个RPC在服务器端成功完成(“我已经发送了我所有的响应!”)，但在客户端失败(“响应在我的截止日期之后到达!”)。服务器也可能在客户端发送所有请求之前决定是否完成。

#### 取消一个RPC

客户端或服务器都可以在任何时候取消RPC。取消操作会立即终止RPC，这样就不会完成进一步的工作。

> 取消之前所做的更改不会回滚

#### 元数据

元数据是关于特定RPC调用的信息(如身份验证细节)，形式为键值对列表，其中键是字符串，值通常是字符串，但也可以是二进制数据。

键由ASCII字母、数字和特殊字符-、\_、组成，不区分大小写。并且不能以grpc-开头(这是为grpc本身保留的)。二进制值的键以-bin结尾，而ascii值的键则不是。

元数据对于gRPC本身是不透明的—它允许客户端提供与服务器调用相关的信息，反之亦然。

对元数据的访问依赖于语言

#### Channels(通道)

gRPC通道用于连接指定主机和端口的gRPC服务器。它在创建客户端存根时使用。客户端可以指定通道参数来修改gRPC的默认行为，例如打开或关闭消息压缩。通道有状态，包括连接状态和空闲状态。

gRPC如何关闭通道取决于语言。有些语言还允许查询通道状态。

## 异步(Async)接口

此节使用 gRPC 的异步/非阻塞 api，用C++编写一个简单的服务器和客户端。
<br/>
gRPC使用 CompletetionQueue API 进行异步操作，基本流程如下：
1. 绑定COmpletionQueue到RPC调用
2. 做一些读/写操作，呈现一个唯一的`void*`标记
3. 调用CompletionQueue::Next来等待操作完成，如果出现标记，则表示响应操作完成。

### 异步客户端

要使用异步客户端调用远程方法，首先创建通道和存根，就像在同步客户端中所做的那样。一旦你有了你的存根，你做以下事情来进行异步调用:

1. 启动RPC并为其创建句柄。将RPC绑定到CompletionQueue
```c++
CompletionQueue cq;
std::unique_ptr<ClientAsyncResponseReader<HelloReply> > rpc(stub_->AsyncSayHello(&context, request, &cq));
```
2. 使用唯一标签询问回复和最终状态
```c++
Status status;
rpc->Finish(&reply, &status, (void*)1);
```
3. 等待完成队列返回下一个标记，一旦返回传递给相应`Finish()`调用的标记，应答和状态就就绪了。
```c++
void* got_tag;
bool ok = false;
cq.Next(&got_tag, &ok);
if (ok && got_tag == (void*)1) {
  // check reply and status
}
```
### 异步服务端

服务器实现请求一个带有标记的RPC调用，然后等待完成队列返回标记。异步处理RPC的基本流程是:

1. 构建导出异步服务的服务器
```c++
helloworld::Greeter::AsyncService service;
ServerBuilder builder;
builder.AddListeningPort("0.0.0.0:50051", InsecureServerCredentials());
builder.RegisterService(&service);
auto cq = builder.AddCompletionQueue();
auto server = builder.BuildAndStart();
```

2. 请求一个RPC，提供一个惟一的标记
```c++
ServerContext context;
HelloRequest request;
ServerAsyncResponseWriter<HelloReply> responder;
service.RequestSayHello(&context, &request, &responder, &cq, &cq, (void*)1);
```

3. 等待完成队列返回标记。一旦检索到标记，上下文、请求和响应程序就准备好了
```c++
HelloReply reply;
Status status;
void* got_tag;
bool ok = false;
cq.Next(&got_tag, &ok);
if (ok && got_tag == (void*)1) {
  // set reply and status
  responder.Finish(reply, status, (void*)2);
}
```

4. 等待完成队列返回标记。返回标签后，RPC就完成了
```c++
void* got_tag;
bool ok = false;
cq.Next(&got_tag, &ok);
if (ok && got_tag == (void*)2) {
  // clean up
}
```

然而，这个基本流没有考虑服务器并发处理多个请求的情况。为了处理这个问题，我们的完整异步服务器示例使用一个CallData对象来维护每个RPC的状态，并使用这个对象的地址作为调用的唯一标记。
```c++
class CallData {
public:
  // Take in the "service" instance (in this case representing an asynchronous
  // server) and the completion queue "cq" used for asynchronous communication
  // with the gRPC runtime.
  CallData(Greeter::AsyncService* service, ServerCompletionQueue* cq)
      : service_(service), cq_(cq), responder_(&ctx_), status_(CREATE) {
    // Invoke the serving logic right away.
    Proceed();
  }

  void Proceed() {
    if (status_ == CREATE) {
      // As part of the initial CREATE state, we *request* that the system
      // start processing SayHello requests. In this request, "this" acts are
      // the tag uniquely identifying the request (so that different CallData
      // instances can serve different requests concurrently), in this case
      // the memory address of this CallData instance.
      service_->RequestSayHello(&ctx_, &request_, &responder_, cq_, cq_,
                                this);
      // Make this instance progress to the PROCESS state.
      status_ = PROCESS;
    } else if (status_ == PROCESS) {
      // Spawn a new CallData instance to serve new clients while we process
      // the one for this CallData. The instance will deallocate itself as
      // part of its FINISH state.
      new CallData(service_, cq_);

      // The actual processing.
      std::string prefix("Hello ");
      reply_.set_message(prefix + request_.name());

      // And we are done! Let the gRPC runtime know we've finished, using the
      // memory address of this instance as the uniquely identifying tag for
      // the event.
      responder_.Finish(reply_, Status::OK, this);
      status_ = FINISH;
    } else {
      GPR_ASSERT(status_ == FINISH);
      // Once in the FINISH state, deallocate ourselves (CallData).
      delete this;
    }
  }
}
```

为简单起见，服务器只对所有事件使用一个完成队列，并在HandleRpcs中运行一个主循环来查询该队列:

```c++
void HandleRpcs() {
  // Spawn a new CallData instance to serve new clients.
  new CallData(&service_, cq_.get());
  void* tag;  // uniquely identifies a request.
  bool ok;
  while (true) {
    // Block waiting to read the next event from the completion queue. The
    // event is uniquely identified by its tag, which in this case is the
    // memory address of a CallData instance.
    cq_->Next(&tag, &ok);
    GPR_ASSERT(ok);
    static_cast<CallData*>(tag)->Proceed();
  }
}
```
### 关闭服务器

我们一直在使用一个完成队列来获得异步通知。在关闭服务器之后，必须小心关闭它。

记住，我们在ServerImpl::Run()中通过运行cq_ = builder.AddCompletionQueue()获得了完成队列实例cq_。看ServerBuilder::AddCompletionQueue的文档，我们看到
> 调用者需要在关闭返回的完成队列之前关闭服务器
更多细节请参考ServerBuilder::AddCompletionQueue的完整文档字符串。在我们的例子中，这意味着ServerImpl的析构函数是这样的:

```c++
~ServerImpl() {
  server_->Shutdown();
  // Always shutdown the completion queue after the server.
  cq_->Shutdown();
}
```

## ALTS 身份验证

c++中使用应用层传输安全(ALTS)的gRPC认证。

### 概述

ALTS (Application Layer Transport Security)是谷歌开发的一种双向认证和传输加密系统。它用于保护谷歌基础架构内的RPC通信。ALTS类似于相互TLS，但经过了设计和优化，以满足谷歌的生产环境的需求。要了解更多信息，请查看ALTS白皮书。

gRPC中的ALTS具有以下特点:
- 创建gRPC服务器和客户端，使用ALTS作为传输安全协议
- ALTS连接端到端受到隐私和完整性保护
- 应用程序可以访问对等信息，如对等服务帐户
- 客户端授权和服务器授权支持
- 最小的代码更改来启用ALTS

gRPC用户可以配置他们的应用程序，使用ALTS作为传输安全协议，只需要几行代码。

> 注意，如果应用程序运行在谷歌云平台上，那么ALTS是完全有效的。通过可插拔的ALTS握手器服务，ALTS可以在任何平台上运行。

### 使用ALTS传输安全协议的gRPC客户端

gRPC客户端可以使用ALTS凭证连接到服务器，如下所示的代码摘录:

```c++
#include <grpcpp/grpcpp.h>
#include <grpcpp/security/credentials.h>

using grpc::experimental::AltsCredentials;
using grpc::experimental::AltsCredentialsOptions;

auto creds = AltsCredentials(AltsCredentialsOptions());
std::shared_ptr<grpc::Channel> channel = CreateChannel(server_address, creds);
```

### 使用ALTS传输安全协议的gRPC服务器

gRPC服务器可以使用ALTS凭证来允许客户端连接到它们，如下图所示:

```c++
#include <grpcpp/security/server_credentials.h>
#include <grpcpp/server.h>
#include <grpcpp/server_builder.h>

using grpc::experimental::AltsServerCredentials;
using grpc::experimental::AltsServerCredentialsOptions;

grpc::ServerBuilder builder;
builder.RegisterService(&service);
auto creds = AltsServerCredentials(AltsServerCredentialsOptions());
builder.AddListeningPort("[::]:<port>", creds);
std::unique_ptr<Server> server(builder.BuildAndStart());
```

### 服务器授权

gRPC内置了使用ALTS的服务器授权支持。使用ALTS的gRPC客户端可以在建立连接之前设置预期的服务器服务帐户。然后，在握手结束时，服务器授权保证服务器标识与客户机指定的服务帐户之一匹配。否则连接失败。

```c++
#include <grpcpp/grpcpp.h>
#include <grpcpp/security/credentials.h>

using grpc::experimental::AltsCredentials;
using grpc::experimental::AltsCredentialsOptions;

AltsCredentialsOptions opts;
opts.target_service_accounts.push_back("expected_server_service_account1");
opts.target_service_accounts.push_back("expected_server_service_account2");
auto creds = AltsCredentials(opts);
std::shared_ptr<grpc::Channel> channel = CreateChannel(server_address, creds);
```

### 客户端授权

在一个成功的连接上，对等信息(例如，客户端的服务帐户)被存储在AltsContext中。gRPC为客户端授权检查提供了一个实用程序库。假设服务器知道预期的客户机标识(例如foo@iam.gserviceaccount.com)，它可以运行以下示例代码来授权传入的RPC。

```c++
#include <grpcpp/server_context.h>
#include <grpcpp/security/alts_util.h>

grpc::ServerContext* context;
grpc::Status status = experimental::AltsClientAuthzCheck(
    context->auth_context(), {"foo@iam.gserviceaccount.com"});
```
