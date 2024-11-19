+++
title = "Generated-code (legacy)"
date = 2024-11-19T10:19:42+08:00
weight = 50
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/languages/go/generated-code-old/](https://grpc.io/docs/languages/go/generated-code-old/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Generated-code (legacy non-generic) reference - 生成代码（非泛型旧版）参考





This page describes the code generated with the [grpc plugin](https://pkg.go.dev/google.golang.org/grpc/cmd/protoc-gen-go-grpc), `protoc-gen-go-grpc`, when compiling `.proto` files with `protoc`.

​	本页面描述了使用 [grpc 插件](https://pkg.go.dev/google.golang.org/grpc/cmd/protoc-gen-go-grpc) `protoc-gen-go-grpc` 编译 `.proto` 文件时生成的代码。

You can find out how to define a gRPC service in a `.proto` file in [Service definition](https://grpc.io/docs/what-is-grpc/core-concepts/#service-definition).

​	您可以在 [服务定义](https://grpc.io/docs/what-is-grpc/core-concepts/#service-definition) 中了解如何在 `.proto` 文件中定义 gRPC 服务。

**Thread-safety**: note that client-side RPC invocations and server-side RPC handlers *are thread-safe* and are meant to be run on concurrent goroutines. But also note that for *individual streams*, incoming and outgoing data is bi-directional but serial; so e.g. *individual streams* do not support *concurrent reads* or *concurrent writes* (but reads are safely concurrent *with* writes).

​	**线程安全性**：注意，客户端 RPC 调用和服务器端 RPC 处理程序是 **线程安全的**，可在并发的 Goroutines 上运行。但需要注意，对于**单个流**，尽管传入和传出数据是双向的，但必须是顺序的。因此，**单个流**不支持**并发读取**或**并发写入**（但读取和写入可以安全地并发执行）。

## 生成的服务器接口上的方法 Methods on generated server interfaces

On the server side, each `service Bar` in the `.proto` file results in the function:

​	在服务器端，每个 `.proto` 文件中的 `service Bar` 会生成以下函数：

```
func RegisterBarServer(s *grpc.Server, srv BarServer)
```

The application can define a concrete implementation of the `BarServer` interface and register it with a `grpc.Server` instance (before starting the server instance) by using this function.

​	应用程序可以定义 `BarServer` 接口的具体实现，并通过此函数在启动服务器实例之前将其注册到 `grpc.Server` 实例中。

### Unary methods

These methods have the following signature on the generated service interface:

​	这些方法在生成的服务接口上的签名如下：

```
Foo(context.Context, *MsgA) (*MsgB, error)
```

In this context, `MsgA` is the protobuf message sent from the client, and `MsgB` is the protobuf message sent back from the server.

​	其中，`MsgA` 是客户端发送的 protobuf 消息，`MsgB` 是服务器返回的 protobuf 消息。

### Server-streaming methods

These methods have the following signature on the generated service interface:

​	这些方法在生成的服务接口上的签名如下：

```
Foo(*MsgA, <ServiceName>_FooServer) error
```

In this context, `MsgA` is the single request from the client, and the `<ServiceName>_FooServer` parameter represents the server-to-client stream of `MsgB` messages.

​	其中，`MsgA` 是客户端发送的单个请求，`<ServiceName>_FooServer` 表示服务器到客户端的 `MsgB` 消息流。

`<ServiceName>_FooServer` has an embedded `grpc.ServerStream` and the following interface:

​	`<ServiceName>_FooServer` 包含一个嵌入的 `grpc.ServerStream`，其接口如下：

```go
type <ServiceName>_FooServer interface {
	Send(*MsgB) error
	grpc.ServerStream
}
```

The server-side handler can send a stream of protobuf messages to the client through this parameter’s `Send` method. End-of-stream for the server-to-client stream is caused by the `return` of the handler method.

​	服务器端处理程序可以通过此参数的 `Send` 方法向客户端发送 protobuf 消息流。处理程序方法的 `return` 表示服务器到客户端流的结束。

### Client-streaming methods

These methods have the following signature on the generated service interface:

​	这些方法在生成的服务接口上的签名如下：

```
Foo(<ServiceName>_FooServer) error
```

In this context, `<ServiceName>_FooServer` can be used both to read the client-to-server message stream and to send the single server response message.

​	其中，`<ServiceName>_FooServer` 可用于读取客户端到服务器的消息流，并发送单个服务器响应消息。

`<ServiceName>_FooServer` has an embedded `grpc.ServerStream` and the following interface:

​	`<ServiceName>_FooServer` 包含一个嵌入的 `grpc.ServerStream`，其接口如下：

```go
type <ServiceName>_FooServer interface {
	SendAndClose(*MsgA) error
	Recv() (*MsgB, error)
	grpc.ServerStream
}
```

The server-side handler can repeatedly call `Recv` on this parameter in order to receive the full stream of messages from the client. `Recv` returns `(nil, io.EOF)` once it has reached the end of the stream. The single response message from the server is sent by calling the `SendAndClose` method on this `<ServiceName>_FooServer` parameter. Note that `SendAndClose` must be called once and only once.

​	服务器端处理程序可以多次调用此参数的 `Recv` 方法，以接收来自客户端的完整消息流。当流结束时，`Recv` 返回 `(nil, io.EOF)`。服务器的单个响应消息通过调用此 `<ServiceName>_FooServer` 参数上的 `SendAndClose` 方法发送。需要注意，`SendAndClose` 必须且只能调用一次。

### Bidi-streaming methods

These methods have the following signature on the generated service interface:

​	这些方法在生成的服务接口上的签名如下：

```
Foo(<ServiceName>_FooServer) error
```

In this context, `<ServiceName>_FooServer` can be used to access both the client-to-server message stream and the server-to-client message stream. `<ServiceName>_FooServer` has an embedded `grpc.ServerStream` and the following interface:

​	`<ServiceName>_FooServer` 可用于访问客户端到服务器的消息流和服务器到客户端的消息流。`<ServiceName>_FooServer` 包含一个嵌入的 `grpc.ServerStream`，其接口如下：

```go
type <ServiceName>_FooServer interface {
	Send(*MsgA) error
	Recv() (*MsgB, error)
	grpc.ServerStream
}
```

The server-side handler can repeatedly call `Recv` on this parameter in order to read the client-to-server message stream. `Recv` returns `(nil, io.EOF)` once it has reached the end of the client-to-server stream. The response server-to-client message stream is sent by repeatedly calling the `Send` method of on this `ServiceName>_FooServer` parameter. End-of-stream for the server-to-client stream is indicated by the `return` of the bidi method handler.

​	服务器端处理程序可以多次调用此参数的 `Recv` 方法以读取客户端到服务器的消息流。当流结束时，`Recv` 返回 `(nil, io.EOF)`。服务器到客户端的响应消息流通过多次调用此 `<ServiceName>_FooServer` 参数的 `Send` 方法发送。服务器到客户端流的结束由双向方法处理程序的 `return` 指示。

## 生成的客户端接口上的方法 Methods on generated client interfaces

For client side usage, each `service Bar` in the `.proto` file also results in the function: `func BarClient(cc *grpc.ClientConn) BarClient`, which returns a concrete implementation of the `BarClient` interface (this concrete implementation also lives in the generated `.pb.go` file).

​	在客户端，每个 `.proto` 文件中的 `service Bar` 都会生成以下函数：`func BarClient(cc *grpc.ClientConn) BarClient`，该函数返回一个 `BarClient` 接口的具体实现（此具体实现也位于生成的 `.pb.go` 文件中）。

### Unary Methods

These methods have the following signature on the generated client stub:

​	这些方法在生成的客户端存根上的签名如下：

```
(ctx context.Context, in *MsgA, opts ...grpc.CallOption) (*MsgB, error)
```

In this context, `MsgA` is the single request from client to server, and `MsgB` contains the response sent back from the server.

​	在此上下文中，`MsgA` 是从客户端到服务器的单个请求，`MsgB` 是服务器返回的响应。

### Server-Streaming methods

These methods have the following signature on the generated client stub:

​	这些方法在生成的客户端存根上的签名如下：

```
Foo(ctx context.Context, in *MsgA, opts ...grpc.CallOption) (<ServiceName>_FooClient, error)
```

In this context, `<ServiceName>_FooClient` represents the server-to-client `stream` of `MsgB` messages.

​	在此上下文中，`<ServiceName>_FooClient` 表示服务器到客户端的 `MsgB` 消息流。

This stream has an embedded `grpc.ClientStream` and the following interface:

​	该流包含一个嵌入的 `grpc.ClientStream`，其接口如下：

```go
type <ServiceName>_FooClient interface {
	Recv() (*MsgB, error)
	grpc.ClientStream
}
```

The stream begins when the client calls the `Foo` method on the stub. The client can then repeatedly call the `Recv` method on the returned `<ServiceName>_FooClient` *stream* in order to read the server-to-client response stream. This `Recv` method returns `(nil, io.EOF)` once the server-to-client stream has been completely read through.

​	当客户端在存根上调用 `Foo` 方法时，流开始。客户端可以反复调用返回的 `<ServiceName>_FooClient` 流上的 `Recv` 方法，以读取服务器到客户端的响应流。当服务器到客户端的流完全读取完时，`Recv` 方法返回 `(nil, io.EOF)`。

### Client-Streaming methods

These methods have the following signature on the generated client stub:

​	这些方法在生成的客户端存根上的签名如下：

```
Foo(ctx context.Context, opts ...grpc.CallOption) (<ServiceName>_FooClient, error)
```

In this context, `<ServiceName>_FooClient` represents the client-to-server `stream` of `MsgA` messages.

​	在此上下文中，`<ServiceName>_FooClient` 表示客户端到服务器的 `MsgA` 消息流。

`<ServiceName>_FooClient` has an embedded `grpc.ClientStream` and the following interface:

​	`<ServiceName>_FooClient` 包含一个嵌入的 `grpc.ClientStream`，其接口如下：

```go
type <ServiceName>_FooClient interface {
	Send(*MsgA) error
	CloseAndRecv() (*MsgB, error)
	grpc.ClientStream
}
```

The stream begins when the client calls the `Foo` method on the stub. The client can then repeatedly call the `Send` method on the returned `<ServiceName>_FooClient` stream in order to send the client-to-server message stream. The `CloseAndRecv` method on this stream must be called once and only once, in order to both close the client-to-server stream and receive the single response message from the server.

​	当客户端在存根上调用 `Foo` 方法时，流开始。客户端可以反复调用返回的 `<ServiceName>_FooClient` 流上的 `Send` 方法，以发送客户端到服务器的消息流。必须且只能调用此流上的 `CloseAndRecv` 方法一次，以同时关闭客户端到服务器的流并接收服务器的单个响应消息。

### Bidi-Streaming methods

These methods have the following signature on the generated client stub:

​	这些方法在生成的客户端存根上的签名如下：

```
Foo(ctx context.Context, opts ...grpc.CallOption) (<ServiceName>_FooClient, error)
```

In this context, `<ServiceName>_FooClient` represents both the client-to-server and server-to-client message streams.

​	在此上下文中，`<ServiceName>_FooClient` 表示客户端到服务器和服务器到客户端的消息流。

`<ServiceName>_FooClient` has an embedded `grpc.ClientStream` and the following interface:

​	`<ServiceName>_FooClient` 包含一个嵌入的 `grpc.ClientStream`，其接口如下：

```go
type <ServiceName>_FooClient interface {
	Send(*MsgA) error
	Recv() (*MsgB, error)
	grpc.ClientStream
}
```

The stream begins when the client calls the `Foo` method on the stub. The client can then repeatedly call the `Send` method on the returned `<SericeName>_FooClient` stream in order to send the client-to-server message stream. The client can also repeatedly call `Recv` on this stream in order to receive the full server-to-client message stream.

​	当客户端在存根上调用 `Foo` 方法时，流开始。客户端可以反复调用返回的 `<ServiceName>_FooClient` 流上的 `Send` 方法，以发送客户端到服务器的消息流。客户端还可以反复调用该流上的 `Recv` 方法，以接收完整的服务器到客户端的消息流。

End-of-stream for the server-to-client stream is indicated by a return value of `(nil, io.EOF)` on the `Recv` method of the stream. End-of-stream for the client-to-server stream can be indicated from the client by calling the `CloseSend` method on the stream.

​	对于服务器到客户端流，流结束由流的 `Recv` 方法返回 `(nil, io.EOF)` 表示。对于客户端到服务器流，客户端可以通过调用流的 `CloseSend` 方法来指示结束。

## Packages and Namespaces

When the `protoc` compiler is invoked with `--go_out=plugins=grpc:`, the `proto package` to Go package translation works the same as when the `protoc-gen-go` plugin is used without the `grpc` plugin.

​	当使用 `--go_out=plugins=grpc:` 调用 `protoc` 编译器时，`proto package` 到 Go 包的转换与不使用 `grpc` 插件的 `protoc-gen-go` 插件相同。

So, for example, if `foo.proto` declares itself to be in `package foo`, then the generated `foo.pb.go` file will also be in the Go `package foo`.

​	例如，如果 `foo.proto` 声明它在 `package foo` 中，则生成的 `foo.pb.go` 文件也将在 Go 的 `package foo` 中。
