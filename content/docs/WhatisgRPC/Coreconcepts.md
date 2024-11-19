+++
title = "核心概念"
date = 2024-11-19T10:19:42+08:00
weight = 10
type = "docs"
description = "gRPC 核心概念简介，以及 gRPC 架构和 RPC 生命周期的概述。"
isCJKLanguage = true
draft = false

+++

> 原文：[https://grpc.io/docs/what-is-grpc/core-concepts/](https://grpc.io/docs/what-is-grpc/core-concepts/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Core concepts, architecture and lifecycle - 核心概念、架构和生命周期

An introduction to key gRPC concepts, with an overview of gRPC architecture and RPC life cycle.



Not familiar with gRPC? First read [Introduction to gRPC]({{< ref "/docs/WhatisgRPC/Introduction" >}}). For language-specific details, see the quick start, tutorial, and reference documentation for your language of choice.

​	如果您不熟悉 gRPC，请先阅读 [gRPC 简介]({{< ref "/docs/WhatisgRPC/Introduction" >}})。有关特定语言的详细信息，请参阅您选择语言的快速入门、教程和参考文档。

{{< youtube "sImWl7JyK_Q">}}

### Overview

#### 服务定义 Service definition

Like many RPC systems, gRPC is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. By default, gRPC uses [protocol buffers](https://developers.google.com/protocol-buffers) as the Interface Definition Language (IDL) for describing both the service interface and the structure of the payload messages. It is possible to use other alternatives if desired.

​	与许多 RPC 系统类似，gRPC 基于定义服务的理念，指定可以远程调用的方法及其参数和返回类型。默认情况下，gRPC 使用 [协议缓冲区](https://developers.google.com/protocol-buffers) 作为接口定义语言（IDL）来描述服务接口和消息负载的结构。根据需要，也可以使用其他替代方案。

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

​	gRPC 支持定义以下四种服务方法：

- Unary RPCs where the client sends a single request to the server and gets a single response back, just like a normal function call.

  - **单向 RPC**：客户端向服务器发送一个请求并收到一个响应，类似普通的函数调用。


  ```proto
  rpc SayHello(HelloRequest) returns (HelloResponse);
  ```

- Server streaming RPCs where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. gRPC guarantees message ordering within an individual RPC call.

  - **服务器流式 RPC**：客户端向服务器发送一个请求并收到一系列消息流作为响应。客户端从返回的流中读取消息，直到没有更多消息为止。gRPC 保证单个 RPC 调用内的消息顺序。


  ```proto
  rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
  ```

- Client streaming RPCs where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them and return its response. Again gRPC guarantees message ordering within an individual RPC call.

  - **客户端流式 RPC**：客户端发送一系列消息给服务器，使用提供的流。客户端完成发送后，等待服务器读取这些消息并返回响应。同样，gRPC 保证单个 RPC 调用内的消息顺序。


  ```proto
  rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
  ```

- Bidirectional streaming RPCs where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved.

  - **双向流式 RPC**：客户端和服务器都使用读写流发送一系列消息。这两个流独立运行，因此客户端和服务器可以按照任意顺序读取和写入。例如，服务器可以在接收完所有客户端消息后写入响应，或者可以交替读取和写入消息，或其他组合。每个流中的消息顺序得以保留。

  ```proto
  rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
  ```


You’ll learn more about the different types of RPC in the [RPC life cycle](https://grpc.io/docs/what-is-grpc/core-concepts/#rpc-life-cycle) section below.

​	关于不同类型的 RPC，请参考下面的 [RPC 生命周期](https://grpc.io/docs/what-is-grpc/core-concepts/#rpc-life-cycle) 部分。

#### Using the API

Starting from a service definition in a `.proto` file, gRPC provides protocol buffer compiler plugins that generate client- and server-side code. gRPC users typically call these APIs on the client side and implement the corresponding API on the server side.

​	从 `.proto` 文件中的服务定义开始，gRPC 提供协议缓冲区编译器插件，用于生成客户端和服务器端代码。gRPC 用户通常在客户端调用这些 API，并在服务器端实现相应的 API。

- On the server side, the server implements the methods declared by the service and runs a gRPC server to handle client calls. The gRPC infrastructure decodes incoming requests, executes service methods, and encodes service responses.
  - **在服务器端**，服务器实现服务声明的方法并运行 gRPC 服务器以处理客户端调用。gRPC 基础设施解码传入请求，执行服务方法并编码服务响应。

- On the client side, the client has a local object known as *stub* (for some languages, the preferred term is *client*) that implements the same methods as the service. The client can then just call those methods on the local object, and the methods wrap the parameters for the call in the appropriate protocol buffer message type, send the requests to the server, and return the server’s protocol buffer responses.
  - **在客户端**，客户端有一个本地对象，称为 *stub*（在某些语言中称为 *client*），它实现与服务相同的方法。客户端只需调用本地对象的方法，这些方法将调用参数封装在适当的协议缓冲区消息类型中，向服务器发送请求，并返回服务器的协议缓冲区响应。

#### Synchronous vs. asynchronous

Synchronous RPC calls that block until a response arrives from the server are the closest approximation to the abstraction of a procedure call that RPC aspires to. On the other hand, networks are inherently asynchronous and in many scenarios it’s useful to be able to start RPCs without blocking the current thread.

​	阻塞直到服务器返回响应的同步 RPC 调用是 RPC 试图抽象出的过程调用的最接近形式。另一方面，网络本质上是异步的，在许多场景下，能够非阻塞地启动 RPC 非常有用。

The gRPC programming API in most languages comes in both synchronous and asynchronous flavors. You can find out more in each language’s tutorial and reference documentation (complete reference docs are coming soon).

​	大多数语言中的 gRPC 编程 API 都提供同步和异步两种形式。您可以在各语言的教程和参考文档中找到更多信息（完整的参考文档即将推出）。

### RPC life cycle

In this section, you’ll take a closer look at what happens when a gRPC client calls a gRPC server method. For complete implementation details, see the language-specific pages.

​	以下部分将深入了解 gRPC 客户端调用 gRPC 服务器方法时发生的情况。完整的实现细节，请参阅特定语言页面。

#### Unary RPC

First consider the simplest type of RPC where the client sends a single request and gets back a single response.

​	先来看最简单的 RPC 类型，客户端发送一个请求并收到一个响应：

1. Once the client calls a stub method, the server is notified that the RPC has been invoked with the client’s [metadata](https://grpc.io/docs/what-is-grpc/core-concepts/#metadata) for this call, the method name, and the specified [deadline](https://grpc.io/docs/what-is-grpc/core-concepts/#deadlines) if applicable. 客户端调用存根方法后，服务器会收到通知，表明此 RPC 调用附带了客户端的 [元数据](https://grpc.io/docs/what-is-grpc/core-concepts/#metadata)、方法名称及指定的 [截止时间](https://grpc.io/docs/what-is-grpc/core-concepts/#deadlines)（如适用）。
2. The server can then either send back its own initial metadata (which must be sent before any response) straight away, or wait for the client’s request message. Which happens first, is application-specific. 服务器可以立即发送初始元数据（必须在任何响应之前发送），也可以等待客户端的请求消息。这取决于具体应用。
3. Once the server has the client’s request message, it does whatever work is necessary to create and populate a response. The response is then returned (if successful) to the client together with status details (status code and optional status message) and optional trailing metadata. 服务器收到客户端请求消息后，执行必要的操作以创建并填充响应。成功时，将响应连同状态详情（状态码和可选的状态消息）及可选的尾随元数据返回给客户端。
4. If the response status is OK, then the client gets the response, which completes the call on the client side. 如果响应状态为 OK，客户端会接收响应，从而完成客户端端的调用。

#### Server streaming RPC

A server-streaming RPC is similar to a unary RPC, except that the server returns a stream of messages in response to a client’s request. After sending all its messages, the server’s status details (status code and optional status message) and optional trailing metadata are sent to the client. This completes processing on the server side. The client completes once it has all the server’s messages.

​	服务器流式 RPC 类似于单向 RPC，但服务器会以流的形式返回一系列消息作为响应。服务器在发送完所有消息后，会将状态详情（状态码和可选状态消息）及可选尾随元数据发送给客户端，从而完成服务器端的处理。客户端在收到服务器的所有消息后完成调用。

#### Client streaming RPC

A client-streaming RPC is similar to a unary RPC, except that the client sends a stream of messages to the server instead of a single message. The server responds with a single message (along with its status details and optional trailing metadata), typically but not necessarily after it has received all the client’s messages.

​	客户端流式 RPC 类似于单向 RPC，但客户端向服务器发送的是一系列消息而不是单一消息。服务器返回一个单一响应（以及状态详情和可选尾随元数据），通常在收到所有客户端消息后。

#### Bidirectional streaming RPC

In a bidirectional streaming RPC, the call is initiated by the client invoking the method and the server receiving the client metadata, method name, and deadline. The server can choose to send back its initial metadata or wait for the client to start streaming messages.

​	在双向流式 RPC 中，调用由客户端通过方法启动，服务器接收客户端元数据、方法名称和截止时间。服务器可以选择发送初始元数据，也可以等待客户端开始流式发送消息。

Client- and server-side stream processing is application specific. Since the two streams are independent, the client and server can read and write messages in any order. For example, a server can wait until it has received all of a client’s messages before writing its messages, or the server and client can play “ping-pong” – the server gets a request, then sends back a response, then the client sends another request based on the response, and so on.

​	客户端和服务器端的流处理是应用程序特定的。由于两个流是独立的，客户端和服务器可以按任意顺序读取和写入。例如，服务器可以在收到所有客户端消息后写入自己的消息，或者客户端和服务器可以交替发送消息，类似“乒乓”的模式。

#### Deadlines/Timeouts

gRPC allows clients to specify how long they are willing to wait for an RPC to complete before the RPC is terminated with a `DEADLINE_EXCEEDED` error. On the server side, the server can query to see if a particular RPC has timed out, or how much time is left to complete the RPC.

​	gRPC 允许客户端指定等待 RPC 完成的最长时间，超出此时间后 RPC 会终止并返回 `DEADLINE_EXCEEDED` 错误。在服务器端，服务器可以查询特定 RPC 是否超时，或查询完成 RPC 的剩余时间。

Specifying a deadline or timeout is language specific: some language APIs work in terms of timeouts (durations of time), and some language APIs work in terms of a deadline (a fixed point in time) and may or may not have a default deadline.

​	指定截止时间或超时是语言特定的：一些语言 API 使用超时（持续时间），一些语言 API 使用截止时间（固定时间点），默认截止时间可能有也可能没有。

#### RPC 终止 RPC termination

In gRPC, both the client and server make independent and local determinations of the success of the call, and their conclusions may not match. This means that, for example, you could have an RPC that finishes successfully on the server side (“I have sent all my responses!”) but fails on the client side (“The responses arrived after my deadline!”). It’s also possible for a server to decide to complete before a client has sent all its requests.

​	在 gRPC 中，客户端和服务器会分别独立地、本地决定调用是否成功，这些结论可能不一致。例如，服务器可能认为 RPC 成功完成（“我已发送所有响应！”），而客户端可能失败（“响应在我的截止时间后才到达！”）。同样，服务器也可能在客户端尚未发送所有请求时决定完成调用。

#### 取消 RPC - Cancelling an RPC

Either the client or the server can cancel an RPC at any time. A cancellation terminates the RPC immediately so that no further work is done.

​	客户端或服务器可以随时取消 RPC。取消操作会立即终止 RPC，不再执行进一步操作。

> Warning
>
> Changes made before a cancellation are not rolled back.
>
> ​	在取消操作之前所做的更改不会回滚。

#### Metadata

Metadata is information about a particular RPC call (such as [authentication details]({{< ref "/docs/Guides/Authentication" >}})) in the form of a list of key-value pairs, where the keys are strings and the values are typically strings, but can be binary data.

​	元数据是关于特定 RPC 调用的信息（例如 [认证详情]({{< ref "/docs/Guides/Authentication" >}})），以键值对列表的形式表示，键是字符串，值通常是字符串，但也可以是二进制数据。

Keys are case insensitive and consist of ASCII letters, digits, and special characters `-`, `_`, `.` and must not start with `grpc-` (which is reserved for gRPC itself). Binary-valued keys end in `-bin` while ASCII-valued keys do not.

​	键不区分大小写，由 ASCII 字母、数字和特殊字符 `-`、`_`、`.` 组成，但不能以 `grpc-` 开头（保留给 gRPC 自身使用）。以 `-bin` 结尾的键是二进制值键，而 ASCII 值键则没有此后缀。

User-defined metadata is not used by gRPC, which allows the client to provide information associated with the call to the server and vice versa.

​	用户定义的元数据不会被 gRPC 使用，这允许客户端向服务器提供与调用相关的信息，反之亦然。

Access to metadata is language dependent.

​	元数据的访问方式取决于具体语言。

#### Channels

A gRPC channel provides a connection to a gRPC server on a specified host and port. It is used when creating a client stub. Clients can specify channel arguments to modify gRPC’s default behavior, such as switching message compression on or off. A channel has state, including `connected` and `idle`.

​	gRPC 通道提供了到指定主机和端口的 gRPC 服务器的连接，用于创建客户端存根。客户端可以指定通道参数以修改 gRPC 的默认行为，例如开启或关闭消息压缩。通道具有状态，包括 `connected` 和 `idle`。

How gRPC deals with closing a channel is language dependent. Some languages also permit querying channel state.

​	gRPC 关闭通道的处理方式取决于具体语言。一些语言还允许查询通道状态。
