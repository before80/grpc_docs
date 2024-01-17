+++
title = "Cancellation"
date = 2024-01-17T08:51:13+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/guides/cancellation/](https://grpc.io/docs/guides/cancellation/)

# Cancellation 取消

Explains how and when to cancel RPCs.

​	说明何时以及如何取消 RPC。



### Overview 概述

When a gRPC client is no longer interested in the result of an RPC call, it may *cancel* to signal this discontinuation of interest to the server. [Deadline]({{< ref "/Guides/Deadlines" >}}) expiration and I/O errors also trigger cancellation. When an RPC is cancelled, the server should stop any ongoing computation and end its side of the stream. Often, servers are also clients to upstream servers, so that cancellation operation should ideally propagate to all ongoing computation in the system that was initiated due to the original client RPC call.

​	当 gRPC 客户端不再对 RPC 调用的结果感兴趣时，它可能会取消以向服务器发出停止关注的信号。截止日期到期和 I/O 错误也会触发取消。当 RPC 被取消时，服务器应停止任何正在进行的计算并结束流的其一端。通常，服务器也是上游服务器的客户端，因此取消操作理想情况下应传播到系统中所有正在进行的计算，这些计算是由于原始客户端 RPC 调用而启动的。

A client may cancel an RPC for several reasons. The data it requested may have been made irrelevant or the author of the client may want to be a good citizen of the server and conserve compute resources.

​	客户端可能出于多种原因取消 RPC。它请求的数据可能变得不相关，或者客户端的作者可能希望成为服务器的好公民并节省计算资源。

```
ClientServer 1Server 2CancelCancelClientServer 1Server 2
```

### Cancelling an RPC Call on the Client Side 在客户端取消 RPC 调用

A client cancels an RPC call by calling a method on the call object or, in some languages, on the accompanying context object. While gRPC clients do not provide additional details to the server about the reason for the cancellation, the cancel API call takes a string describing the reason, which will result in a client-side exception and/or log containing the provided reason. When a server is notified of the cancellation of an RPC, the application-provided server handler may be busy processing the request. The gRPC library in general does not have a mechanism to interrupt the application-provided server handler, so the server handler must coordinate with the gRPC library to ensure that local processing of the request ceases. Therefore, if an RPC is long-lived, its server handler must periodically check if the RPC it is servicing has been cancelled and if it has, cease processing. Some languages will also support automatic cancellation of anyoutgoing RPCs, while in others, the author of the server handler is responsible for this.

​	客户端通过调用调用对象上的方法或在某些语言中调用随附的上下文对象上的方法来取消 RPC 调用。虽然 gRPC 客户端不会向服务器提供有关取消原因的其他详细信息，但取消 API 调用会采用描述原因的字符串，这将导致包含所提供原因的客户端异常和/或日志。当服务器收到 RPC 取消的通知时，应用程序提供的服务器处理程序可能正忙于处理请求。gRPC 库通常没有机制来中断应用程序提供的服务器处理程序，因此服务器处理程序必须与 gRPC 库协调，以确保请求的本地处理停止。因此，如果 RPC 持续时间较长，则其服务器处理程序必须定期检查它正在服务的 RPC 是否已取消，如果已取消，则停止处理。某些语言还将支持自动取消任何传出 RPC，而在其他语言中，服务器处理程序的编写者负责此操作。

```
CANCEL
CANCEL
Server1
false

true

perform some work
cancelled?
cancel upstream RPCs
exit RPC handler
Client
Server2
```

### Language Support 语言支持

| Language 语言 | Example 示例                                                 | Notes 备注                                           |
| ------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| Java          | [Example 示例](https://github.com/grpc/grpc-java/tree/master/examples/src/main/java/io/grpc/examples/cancellation) | Automatically cancels outgoing RPCs 自动取消传出 RPC |
| Go            | [Example 示例](https://github.com/grpc/grpc-go/tree/master/examples/features/cancellation) | Automatically cancels outgoing RPCs 自动取消传出 RPC |
| C++           | [Example 示例](https://github.com/grpc/grpc/tree/master/examples/cpp/cancellation) | Automatically cancels outgoing RPCs 自动取消传出 RPC |
| Python        | [Example 示例](https://github.com/grpc/grpc/tree/master/examples/python/cancellation) |                                                      |
