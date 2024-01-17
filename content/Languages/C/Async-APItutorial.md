+++
title = "Async-API tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/cpp/async/](https://grpc.io/docs/languages/cpp/async/)

# Asynchronous-API tutorial 异步 API 教程





This tutorial shows you how to write a simple server and client in C++ using gRPC’s asynchronous/non-blocking APIs. It assumes you are already familiar with writing simple synchronous gRPC code, as described in [Basics tutorial]({{< ref "/Languages/C/Basicstutorial" >}}). The example used in this tutorial follows from the basic [Greeter example](https://github.com/grpc/grpc/tree/v1.60.0/examples/cpp/helloworld) used in the [quick start]({{< ref "/Languages/C/Quickstart" >}}). You’ll find it along with installation instructions in [grpc/examples/cpp/helloworld](https://github.com/grpc/grpc/tree/v1.60.0/examples/cpp/helloworld).

​	本教程演示了如何使用 gRPC 的异步/非阻塞 API 在 C++ 中编写简单的服务器和客户端。它假定您已经熟悉编写简单的同步 gRPC 代码，如基本教程中所述。本教程中使用的示例遵循快速入门中使用的基本 Greeter 示例。您可以在 grpc/examples/cpp/helloworld 中找到它以及安装说明。

### Overview 概述

gRPC uses the [CompletionQueue](https://grpc.io/grpc/cpp/classgrpc_1_1_completion_queue.html) API for asynchronous operations. The basic work flow is as follows:

​	gRPC 使用 CompletionQueue API 进行异步操作。基本工作流程如下：

- bind a `CompletionQueue` to an RPC call
  将 `CompletionQueue` 绑定到 RPC 调用
- do something like a read or write, present with a unique `void*` tag
  执行读取或写入等操作，并提供唯一的 `void*` 标记
- call `CompletionQueue::Next` to wait for operations to complete. If a tag appears, it indicates that the corresponding operation is complete.
  调用 `CompletionQueue::Next` 以等待操作完成。如果出现标记，则表示相应的操作已完成。

### Async client 异步客户端

To use an asynchronous client to call a remote method, you first create a channel and stub, just as you do in a [synchronous client](https://github.com/grpc/grpc/blob/v1.60.0/examples/cpp/helloworld/greeter_client.cc). Once you have your stub, you do the following to make an asynchronous call:

​	要使用异步客户端调用远程方法，首先要创建一个通道和存根，就像在同步客户端中所做的那样。拥有存根后，可以执行以下操作以进行异步调用：

- Initiate the RPC and create a handle for it. Bind the RPC to a `CompletionQueue`.

  ​	启动 RPC 并为此创建一个句柄。将 RPC 绑定到 `CompletionQueue` 。

  ```c
  CompletionQueue cq;
  std::unique_ptr<ClientAsyncResponseReader<HelloReply> > rpc(
      stub_->AsyncSayHello(&context, request, &cq));
  ```

- Ask for the reply and final status, with a unique tag

  ​	使用唯一标记请求回复和最终状态

  ```c
  Status status;
  rpc->Finish(&reply, &status, (void*)1);
  ```

- Wait for the completion queue to return the next tag. The reply and status are ready once the tag passed into the corresponding `Finish()` call is returned.

  ​	等待完成队列返回下一个标记。一旦将标记传递到相应的 `Finish()` 调用中，回复和状态即会准备就绪。

  ```c
  void* got_tag;
  bool ok = false;
  cq.Next(&got_tag, &ok);
  if (ok && got_tag == (void*)1) {
    // check reply and status
  }
  ```

You can see the complete client example in [greeter_async_client.cc](https://github.com/grpc/grpc/blob/v1.60.0/examples/cpp/helloworld/greeter_async_client.cc).

​	您可以在 greeter_async_client.cc 中看到完整的客户端示例。

### Async server 异步服务器

The server implementation requests an RPC call with a tag and then waits for the completion queue to return the tag. The basic flow for handling an RPC asynchronously is:

​	服务器实现请求带有标记的 RPC 调用，然后等待完成队列返回该标记。异步处理 RPC 的基本流程如下：

- Build a server exporting the async service

  ​	构建导出异步服务的服务器

  ```c
  helloworld::Greeter::AsyncService service;
  ServerBuilder builder;
  builder.AddListeningPort("0.0.0.0:50051", InsecureServerCredentials());
  builder.RegisterService(&service);
  auto cq = builder.AddCompletionQueue();
  auto server = builder.BuildAndStart();
  ```

- Request one RPC, providing a unique tag

  ​	请求一个 RPC，提供一个唯一标记

  ```c
  ServerContext context;
  HelloRequest request;
  ServerAsyncResponseWriter<HelloReply> responder;
  service.RequestSayHello(&context, &request, &responder, &cq, &cq, (void*)1);
  ```

- Wait for the completion queue to return the tag. The context, request and responder are ready once the tag is retrieved.

  ​	等待完成队列返回该标记。一旦检索到标记，上下文、请求和响应者就准备好了。

  ```c
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

- Wait for the completion queue to return the tag. The RPC is finished when the tag is back.

  ​	等待完成队列返回该标记。当标记返回时，RPC 完成。

  ```c
  void* got_tag;
  bool ok = false;
  cq.Next(&got_tag, &ok);
  if (ok && got_tag == (void*)2) {
    // clean up
  }
  ```

This basic flow, however, doesn’t take into account the server handling multiple requests concurrently. To deal with this, our complete async server example uses a `CallData` object to maintain the state of each RPC, and uses the address of this object as the unique tag for the call.

​	但是，此基本流程并未考虑服务器同时处理多个请求的情况。为了解决这个问题，我们的完整异步服务器示例使用 `CallData` 对象来维护每个 RPC 的状态，并使用此对象的地址作为调用的唯一标记。

```c
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

For simplicity the server only uses one completion queue for all events, and runs a main loop in `HandleRpcs` to query the queue:

​	为了简单起见，服务器仅对所有事件使用一个完成队列，并在 `HandleRpcs` 中运行一个主循环来查询队列：

```c
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

#### Shutting Down the Server 关闭服务器

We’ve been using a completion queue to get the async notifications. Care must be taken to shut it down *after* the server has also been shut down.

​	我们一直在使用完成队列来获取异步通知。在服务器关闭后，必须小心将其关闭。

Remember we got our completion queue instance `cq_` in `ServerImpl::Run()` by running `cq_ = builder.AddCompletionQueue()`. Looking at `ServerBuilder::AddCompletionQueue`’s documentation we see that

​	记得我们在 `ServerImpl::Run()` 中通过运行 `cq_ = builder.AddCompletionQueue()` 获取了完成队列实例 `cq_` 。查看 `ServerBuilder::AddCompletionQueue` 的文档，我们看到

> … Caller is required to shutdown the server prior to shutting down the returned completion queue.
>
> ​	… 调用者需要在关闭返回的完成队列之前关闭服务器。

Refer to `ServerBuilder::AddCompletionQueue`’s full docstring for more details. What this means in our example is that `ServerImpl's` destructor looks like:

​	有关更多详细信息，请参阅 `ServerBuilder::AddCompletionQueue` 的完整文档字符串。在我们的示例中，这意味着 `ServerImpl's` 的析构函数如下所示：

```c
~ServerImpl() {
  server_->Shutdown();
  // Always shutdown the completion queue after the server.
  cq_->Shutdown();
}
```

You can see our complete server example in [greeter_async_server.cc](https://github.com/grpc/grpc/blob/v1.60.0/examples/cpp/helloworld/greeter_async_server.cc).

​	您可以在 greeter_async_server.cc 中看到我们完整的服务器示例。
