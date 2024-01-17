+++
title = "Core concepts"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/what-is-grpc/core-concepts/](https://grpc.io/docs/what-is-grpc/core-concepts/)

# Core concepts, architecture and lifecycle 核心概念、架构和生命周期

An introduction to key gRPC concepts, with an overview of gRPC architecture and RPC life cycle.

​	介绍 gRPC 的主要概念，并概述 gRPC 架构和 RPC 生命周期。



Not familiar with gRPC? First read [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}). For language-specific details, see the quick start, tutorial, and reference documentation for your language of choice.

​	不熟悉 gRPC？首先阅读 gRPC 简介。有关特定语言的详细信息，请参阅您选择的语言的快速入门、教程和参考文档。

### Overview 概述

#### Service definition 服务定义

Like many RPC systems, gRPC is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. By default, gRPC uses [protocol buffers](https://developers.google.com/protocol-buffers) as the Interface Definition Language (IDL) for describing both the service interface and the structure of the payload messages. It is possible to use other alternatives if desired.

​	与许多 RPC 系统一样，gRPC 基于定义服务的想法，指定可通过其参数和返回类型远程调用的方法。默认情况下，gRPC 使用协议缓冲区作为接口定义语言 (IDL)，用于描述服务接口和有效负载消息的结构。如果需要，可以使用其他替代方案。

```proto
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

gRPC lets you define four kinds of service method:

​	gRPC 允许您定义四种服务方法：

- Unary RPCs where the client sends a single request to the server and gets a single response back, just like a normal function call.

  ​	单一 RPC，其中客户端向服务器发送单个请求并获得单个响应，就像普通函数调用一样。

  ```proto
  rpc SayHello(HelloRequest) returns (HelloResponse);
  ```

- Server streaming RPCs where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. gRPC guarantees message ordering within an individual RPC call.

  ​	服务器流式 RPC，其中客户端向服务器发送请求并获取流以读取一系列消息。客户端从返回的流中读取，直到没有更多消息。gRPC 保证在单个 RPC 调用中对消息进行排序。

  ```proto
  rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
  ```

- Client streaming RPCs where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them and return its response. Again gRPC guarantees message ordering within an individual RPC call.

  ​	客户端流式 RPC，其中客户端编写一系列消息并使用提供的流将它们发送到服务器。客户端完成编写消息后，它会等待服务器读取它们并返回其响应。同样，gRPC 保证在单个 RPC 调用中对消息进行排序。

  ```proto
  rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
  ```

- Bidirectional streaming RPCs where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved.

  ​	双方使用读写流发送一系列消息的双向流式 RPC。两个流独立运行，因此客户端和服务器可以按照他们喜欢的任何顺序进行读写：例如，服务器可以等到收到所有客户端消息后再写出其响应，或者它可以交替读取消息然后写出消息，或者其他一些读写组合。每个流中的消息顺序都将被保留。

  ```proto
  rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
  ```

You’ll learn more about the different types of RPC in the [RPC life cycle]({{< ref "/WhatisgRPC/Coreconcepts#rpc-life-cycle" >}}) section below.

​	您将在下面的 RPC 生命周期部分中了解有关不同类型 RPC 的更多信息。

#### Using the API 使用 API

Starting from a service definition in a `.proto` file, gRPC provides protocol buffer compiler plugins that generate client- and server-side code. gRPC users typically call these APIs on the client side and implement the corresponding API on the server side.

​	从 `.proto` 文件中的服务定义开始，gRPC 提供生成客户端和服务器端代码的协议缓冲区编译器插件。gRPC 用户通常在客户端调用这些 API，并在服务器端实现相应的 API。

- On the server side, the server implements the methods declared by the service and runs a gRPC server to handle client calls. The gRPC infrastructure decodes incoming requests, executes service methods, and encodes service responses.
  在服务器端，服务器实现服务声明的方法并运行 gRPC 服务器来处理客户端调用。gRPC 基础架构解码传入请求、执行服务方法并对服务响应进行编码。
- On the client side, the client has a local object known as *stub* (for some languages, the preferred term is *client*) that implements the same methods as the service. The client can then just call those methods on the local object, and the methods wrap the parameters for the call in the appropriate protocol buffer message type, send the requests to the server, and return the server’s protocol buffer responses.
  在客户端，客户端有一个称为存根的本地对象（对于某些语言，首选术语是客户端），它实现了与服务相同的方法。然后，客户端只需在本地对象上调用这些方法，这些方法就会将调用参数包装在适当的协议缓冲区消息类型中，将请求发送到服务器，并返回服务器的协议缓冲区响应。

#### Synchronous vs. asynchronous 同步与异步

Synchronous RPC calls that block until a response arrives from the server are the closest approximation to the abstraction of a procedure call that RPC aspires to. On the other hand, networks are inherently asynchronous and in many scenarios it’s useful to be able to start RPCs without blocking the current thread.

​	同步 RPC 调用会一直阻塞，直到从服务器收到响应，这是 RPC 想要实现的接近过程调用抽象的最佳方式。另一方面，网络本质上是异步的，在许多情况下，能够在不阻塞当前线程的情况下启动 RPC 会很有用。

The gRPC programming API in most languages comes in both synchronous and asynchronous flavors. You can find out more in each language’s tutorial and reference documentation (complete reference docs are coming soon).

​	大多数语言中的 gRPC 编程 API 既有同步版本，也有异步版本。您可以在每种语言的教程和参考文档中找到更多信息（完整的参考文档即将推出）。

### RPC life cycle RPC 生命周期

In this section, you’ll take a closer look at what happens when a gRPC client calls a gRPC server method. For complete implementation details, see the language-specific pages.

​	在本节中，您将详细了解 gRPC 客户端调用 gRPC 服务器方法时发生的情况。有关完整的实现详细信息，请参阅特定于语言的页面。

#### Unary RPC 一元 RPC

First consider the simplest type of RPC where the client sends a single request and gets back a single response.

​	首先考虑最简单的 RPC 类型，其中客户端发送单个请求并获得单个响应。

1. Once the client calls a stub method, the server is notified that the RPC has been invoked with the client’s [metadata]({{< ref "/WhatisgRPC/Coreconcepts#metadata" >}}) for this call, the method name, and the specified [deadline]({{< ref "/WhatisgRPC/Coreconcepts#deadlines" >}}) if applicable.
   客户端调用存根方法后，服务器会收到通知，告知已调用 RPC，其中包含此调用的客户端元数据、方法名称以及适用的指定截止时间。
2. The server can then either send back its own initial metadata (which must be sent before any response) straight away, or wait for the client’s request message. Which happens first, is application-specific.
   然后，服务器可以立即发回自己的初始元数据（必须在任何响应之前发送），或等待客户端的请求消息。哪种情况先发生取决于具体应用。
3. Once the server has the client’s request message, it does whatever work is necessary to create and populate a response. The response is then returned (if successful) to the client together with status details (status code and optional status message) and optional trailing metadata.
   服务器收到客户端的请求消息后，会执行必要的工作来创建和填充响应。然后将响应（如果成功）连同状态详细信息（状态代码和可选状态消息）和可选尾部元数据一起返回给客户端。
4. If the response status is OK, then the client gets the response, which completes the call on the client side.
   如果响应状态为“确定”，则客户端会收到响应，从而完成客户端上的调用。

#### Server streaming RPC 服务器流式 RPC

A server-streaming RPC is similar to a unary RPC, except that the server returns a stream of messages in response to a client’s request. After sending all its messages, the server’s status details (status code and optional status message) and optional trailing metadata are sent to the client. This completes processing on the server side. The client completes once it has all the server’s messages.

​	服务器流式 RPC 与一元 RPC 类似，不同之处在于服务器返回一个消息流来响应客户端的请求。在发送完所有消息后，服务器的状态详细信息（状态代码和可选状态消息）和可选的尾部元数据将发送给客户端。这完成了服务器端的处理。客户端在收到服务器的所有消息后完成处理。

#### Client streaming RPC 客户端流式 RPC

A client-streaming RPC is similar to a unary RPC, except that the client sends a stream of messages to the server instead of a single message. The server responds with a single message (along with its status details and optional trailing metadata), typically but not necessarily after it has received all the client’s messages.

​	客户端流式 RPC 与一元 RPC 类似，不同之处在于客户端向服务器发送一个消息流，而不是一个消息。服务器使用单个消息（以及其状态详细信息和可选的尾部元数据）进行响应，通常（但不一定）在收到客户端的所有消息后进行响应。

#### Bidirectional streaming RPC 双向流式 RPC

In a bidirectional streaming RPC, the call is initiated by the client invoking the method and the server receiving the client metadata, method name, and deadline. The server can choose to send back its initial metadata or wait for the client to start streaming messages.

​	在双向流式 RPC 中，调用由客户端发起，客户端调用方法，服务器接收客户端元数据、方法名称和截止时间。服务器可以选择发回其初始元数据或等待客户端开始流式传输消息。

Client- and server-side stream processing is application specific. Since the two streams are independent, the client and server can read and write messages in any order. For example, a server can wait until it has received all of a client’s messages before writing its messages, or the server and client can play “ping-pong” – the server gets a request, then sends back a response, then the client sends another request based on the response, and so on.

​	客户端和服务器端的流处理是特定于应用程序的。由于两个流是独立的，因此客户端和服务器可以按任何顺序读取和写入消息。例如，服务器可以等到收到客户端的所有消息后再写入其消息，或者服务器和客户端可以玩“乒乓球”——服务器收到请求，然后发回响应，然后客户端根据响应发送另一个请求，依此类推。

#### Deadlines/Timeouts 截止时间/超时

gRPC allows clients to specify how long they are willing to wait for an RPC to complete before the RPC is terminated with a `DEADLINE_EXCEEDED` error. On the server side, the server can query to see if a particular RPC has timed out, or how much time is left to complete the RPC.

​	gRPC 允许客户端指定他们愿意等待 RPC 完成多长时间，然后在 RPC 以 `DEADLINE_EXCEEDED` 错误终止之前终止 RPC。在服务器端，服务器可以查询以查看特定 RPC 是否已超时，或完成 RPC 还剩余多少时间。

Specifying a deadline or timeout is language specific: some language APIs work in terms of timeouts (durations of time), and some language APIs work in terms of a deadline (a fixed point in time) and may or may not have a default deadline.

​	指定截止时间或超时时间取决于语言：某些语言 API 使用超时时间（持续时间），而某些语言 API 使用截止时间（固定时间点），并且可能没有默认截止时间。

#### RPC termination RPC 终止

In gRPC, both the client and server make independent and local determinations of the success of the call, and their conclusions may not match. This means that, for example, you could have an RPC that finishes successfully on the server side (“I have sent all my responses!”) but fails on the client side (“The responses arrived after my deadline!”). It’s also possible for a server to decide to complete before a client has sent all its requests.

​	在 gRPC 中，客户端和服务器都会独立且本地地确定调用的成功与否，并且它们的结论可能不一致。这意味着，例如，您可能有一个在服务器端成功完成的 RPC（“我已经发送了所有响应！”），但在客户端端失败（“响应在我的截止时间之后到达！”）。服务器也有可能在客户端发送所有请求之前决定完成。

#### Cancelling an RPC 取消 RPC

Either the client or the server can cancel an RPC at any time. A cancellation terminates the RPC immediately so that no further work is done.

​	客户端或服务器可以随时取消 RPC。取消会立即终止 RPC，以便不再执行进一步的工作。

#### Warning 警告

Changes made before a cancellation are not rolled back.
在取消之前所做的更改不会回滚。

#### Metadata 元数据

Metadata is information about a particular RPC call (such as [authentication details]({{< ref "/Guides/Authentication" >}})) in the form of a list of key-value pairs, where the keys are strings and the values are typically strings, but can be binary data.

​	元数据是有关特定 RPC 调用的信息（例如身份验证详细信息），采用键值对列表的形式，其中键是字符串，值通常是字符串，但可以是二进制数据。

Keys are case insensitive and consist of ASCII letters, digits, and special characters `-`, `_`, `.` and must not start with `grpc-` (which is reserved for gRPC itself). Binary-valued keys end in `-bin` while ASCII-valued keys do not.

​	键不区分大小写，由 ASCII 字母、数字和特殊字符 `-` 、 `_` 、 `.` 组成，且不能以 `grpc-` 开头（该字符专用于 gRPC 本身）。二进制值键以 `-bin` 结尾，而 ASCII 值键则不以 `-bin` 结尾。

User-defined metadata is not used by gRPC, which allows the client to provide information associated with the call to the server and vice versa.

​	用户定义的元数据不会被 gRPC 使用，这允许客户端向服务器提供与调用相关的信息，反之亦然。

Access to metadata is language dependent.

​	访问元数据取决于语言。

#### Channels 通道

A gRPC channel provides a connection to a gRPC server on a specified host and port. It is used when creating a client stub. Clients can specify channel arguments to modify gRPC’s default behavior, such as switching message compression on or off. A channel has state, including `connected` and `idle`.

​	gRPC 通道提供与指定主机和端口上的 gRPC 服务器的连接。在创建客户端存根时使用它。客户端可以指定通道参数来修改 gRPC 的默认行为，例如打开或关闭消息压缩。通道具有状态，包括 `connected` 和 `idle` 。

How gRPC deals with closing a channel is language dependent. Some languages also permit querying channel state.

​	gRPC 如何处理关闭通道取决于语言。某些语言还允许查询通道状态。
