+++
title = "生成代码"
date = 2024-11-19T10:19:42+08:00
weight = 40
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

> 原文：[https://grpc.io/docs/languages/go/generated-code/](https://grpc.io/docs/languages/go/generated-code/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Generated-code reference - 生成代码参考





This page describes the code generated when compiling `.proto` files with `protoc`, using the `protoc-gen-go-grpc` [grpc plugin](https://pkg.go.dev/google.golang.org/grpc/cmd/protoc-gen-go-grpc). This latest version of generated code uses generics by default. If you’re working with older generated code that doesn’t use generics, you can find the relevant documentation [here]({{< ref "/docs/Languages/Go/Generated-codelegacy" >}}). While we encourage using this latest version with generics, you can temporarily revert to the old behavior by setting the `useGenericStreams` flag to `false`.

​	本页面描述了使用 `protoc-gen-go-grpc` [gRPC 插件](https://pkg.go.dev/google.golang.org/grpc/cmd/protoc-gen-go-grpc) 编译 `.proto` 文件时生成的代码。最新版本的生成代码默认使用泛型。如果您使用的是不支持泛型的旧版生成代码，可以在 [此处]({{< ref "/docs/Languages/Go/Generated-codelegacy" >}}) 找到相关文档。我们建议使用支持泛型的最新版本，但可以通过将 `useGenericStreams` 标志设置为 `false` 暂时恢复旧版行为。

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
Foo(context.Context, *RequestMsg) (*ResponseMsg, error)
```

In this context, `RequestMsg` is the protobuf message sent from the client, and `ResponseMsg` is the protobuf message sent back from the server.

​	其中，`RequestMsg` 是客户端发送的 protobuf 消息，`ResponseMsg` 是服务器返回的 protobuf 消息。

### Server-streaming methods

These methods have the following signature on the generated service interface: 

​	这些方法在生成的服务接口上的签名如下：

```
Foo(*RequestMsg, grpc.ServerStreamingServer[*ResponseMsg]) error
```

In this context, `RequestMsg` is the single request from the client, and [`grpc.ServerStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#ServerStreamingServer) represents the server side of server-to-client stream of response type `ResponseMsg`.

​	其中，`RequestMsg` 是客户端发送的单个请求，[`grpc.ServerStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#ServerStreamingServer) 表示服务器端的响应类型为 `ResponseMsg` 的服务器到客户端流。

Refer to [`grpc.ServerStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#ServerStreamingServer) documentation for detailed usage information.

​	有关详细用法，请参考 [`grpc.ServerStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#ServerStreamingServer) 文档。

### Client-streaming methods

These methods have the following signature on the generated service interface:

​	这些方法在生成的服务接口上的签名如下：

```
Foo(grpc.ClientStreamingServer[*RequestMsg, *ResponseMsg]) error
```

Where `RequestMsg` is the message type of the stream, sent from client-to-server and `ResponseMsg` is the type of response from server to client.

​	其中，`RequestMsg` 是从客户端到服务器的消息流的消息类型，`ResponseMsg` 是从服务器到客户端的单个响应的类型。

In this context, [`grpc.ClientStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#ClientStreamingServer) can be used both to read the client-to-server message stream and to send the single server response message.

​	[`grpc.ClientStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#ClientStreamingServer) 可用于读取客户端到服务器的消息流，并发送单个服务器响应消息。

Refer to [`grpc.ClientStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#ClientStreamingServer) documentation for detailed usage information.

​	有关详细用法，请参考 [`grpc.ClientStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#ClientStreamingServer) 文档。

### 双向流式方法 Bidi-streaming methods

These methods have the following signature on the generated service interface:

​	这些方法在生成的服务接口上的签名如下：

```
Foo(grpc.BidiStreamingServer[*RequestMsg, *ResponseMsg]) error
```

Where `RequestMsg` is the message type of the stream, sent from client-to-server and `ResponseMsg` is the type of stream from server-to-client.

​	其中，`RequestMsg` 是从客户端到服务器的消息流的消息类型，`ResponseMsg` 是从服务器到客户端的消息流的消息类型。

In this context, [`grpc.BidiStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#BidiStreamingServer) can be used to access both the client-to-server message stream and the server-to-client message stream.

​	[`grpc.BidiStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#BidiStreamingServer) 可用于访问客户端到服务器的消息流和服务器到客户端的消息流。

Refer to [`grpc.BidiStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#BidiStreamingServer) documentation for detailed usage information.

​	有关详细用法，请参考 [`grpc.BidiStreamingServer`](https://pkg.go.dev/google.golang.org/grpc#BidiStreamingServer) 文档。

## 生成的客户端接口上的方法 Methods on generated client interfaces

For client side usage, each `service Bar` in the `.proto` file also results in the function: `func BarClient(cc *grpc.ClientConn) BarClient`, which returns a concrete implementation of the `BarClient` interface (this concrete implementation also lives in the generated `.pb.go` file).

​	在客户端，每个 `.proto` 文件中的 `service Bar` 还会生成以下函数：`func BarClient(cc *grpc.ClientConn) BarClient`，该函数返回一个 `BarClient` 接口的具体实现（此具体实现也位于生成的 `.pb.go` 文件中）。

### Unary Methods

These methods have the following signature on the generated client stub:

​	这些方法在生成的客户端存根上的签名如下：

```
(ctx context.Context, in *RequestMsg, opts ...grpc.CallOption) (*ResponseMsg, error)
```

In this context, `RequestMsg` is the single request from client to server, and `ResponseMsg` contains the response sent back from the server.

​	在此上下文中，`RequestMsg` 是从客户端到服务器的单个请求，`ResponseMsg` 包含服务器返回的响应。

### Server-Streaming methods

These methods have the following signature on the generated client stub:

​	这些方法在生成的客户端存根上的签名如下：

```
Foo(ctx context.Context, in *RequestMsg, opts ...grpc.CallOption) (grpc.ServerStreamingClient[*ResponseMsg], error)
```

In this context, [`grpc.ServerStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#ServerStreamingClient) represents the client side of server-to-client stream of `ResponseMsg` messages.

​	在此上下文中，[`grpc.ServerStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#ServerStreamingClient) 表示从服务器到客户端的 `ResponseMsg` 消息流的客户端端。

Refer to [`grpc.ServerStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#ServerStreamingClient) documentation for detailed usage information.

​	有关详细用法，请参考 [`grpc.ServerStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#ServerStreamingClient) 文档。

### Client-Streaming methods

These methods have the following signature on the generated client stub:

​	这些方法在生成的客户端存根上的签名如下：

```
Foo(ctx context.Context, opts ...grpc.CallOption) (grpc.ClientStreamingClient[*RequestMsg, *ResponseMsg], error)
```

In this context, [`grpc.ClientStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#ClientStreamingClient) represents the client side of client-to-server stream of `RequestMsg` messages. It can be used both to send the client-to-server message stream and to receive the single server response message.

​	在此上下文中，[`grpc.ClientStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#ClientStreamingClient) 表示从客户端到服务器的 `RequestMsg` 消息流的客户端端。它可用于发送客户端到服务器的消息流并接收服务器的单个响应消息。

Refer to [`grpc.ClientStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#ClientStreamingClient) documentation for detailed usage information.

​	有关详细用法，请参考 [`grpc.ClientStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#ClientStreamingClient) 文档。

### Bidi-Streaming methods

These methods have the following signature on the generated client stub:

​	这些方法在生成的客户端存根上的签名如下：

```
Foo(ctx context.Context, opts ...grpc.CallOption) (grpc.BidiStreamingClient[*RequestMsg, *ResponseMsg], error)
```

In this context, [`grpc.BidiStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#BidiStreamingClient) represents both the client-to-server and server-to-client message streams.

​	在此上下文中，[`grpc.BidiStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#BidiStreamingClient) 表示客户端到服务器和服务器到客户端的消息流。

Refer to [`grpc.BidiStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#BidiStreamingClient) documentation for detailed usage information.

​	有关详细用法，请参考 [`grpc.BidiStreamingClient`](https://pkg.go.dev/google.golang.org/grpc#BidiStreamingClient) 文档。

## Packages and Namespaces

When the `protoc` compiler is invoked with `--go_out=plugins=grpc:`, the `proto package` to Go package translation works the same as when the `protoc-gen-go` plugin is used without the `grpc` plugin.

​	当使用 `--go_out=plugins=grpc:` 调用 `protoc` 编译器时，`proto package` 到 Go 包的转换与不使用 `grpc` 插件的 `protoc-gen-go` 插件相同。

So, for example, if `foo.proto` declares itself to be in `package foo`, then the generated `foo.pb.go` file will also be in the Go `package foo`.

​	例如，如果 `foo.proto` 声明它在 `package foo` 中，则生成的 `foo.pb.go` 文件也将在 Go 的 `package foo` 中。
