+++
title = "Quick start"
date = 2024-11-19T10:19:42+08:00
weight = 1
type = "docs"
description = "本指南将通过一个简单的示例帮助您快速上手 Go 中的 gRPC。"
isCJKLanguage = true
draft = false

+++

> 原文：[https://grpc.io/docs/languages/go/quickstart/](https://grpc.io/docs/languages/go/quickstart/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Quick start

This guide gets you started with gRPC in Go with a simple working example.

​	本指南将通过一个简单的示例帮助您快速上手 Go 中的 gRPC。

### 前置条件 Prerequisites

- **[Go](https://golang.org/)**, any one of the **two latest major** [releases of Go](https://golang.org/doc/devel/release.html).

  - **[Go](https://golang.org/)**：需要安装 [Go](https://golang.org/doc/devel/release.html) 的最新两个主要版本之一。


  For installation instructions, see Go’s [Getting Started](https://golang.org/doc/install) guide.

  ​	有关安装说明，请参阅 Go 的 [快速入门指南](https://golang.org/doc/install)。

- **[Protocol buffer](https://developers.google.com/protocol-buffers) compiler**, `protoc`, [version 3](https://protobuf.dev/programming-guides/proto3).

  - **[协议缓冲区](https://developers.google.com/protocol-buffers) 编译器**：`protoc`，[版本 3](https://protobuf.dev/programming-guides/proto3)。


  For installation instructions, see [Protocol Buffer Compiler Installation](https://grpc.io/docs/protoc-installation/).

  ​	有关安装说明，请参阅 [协议缓冲区编译器安装指南](https://grpc.io/docs/protoc-installation/)。

- **Go plugins** for the protocol compiler: **协议编译器的 Go 插件**：

  1. Install the protocol compiler plugins for Go using the following commands: 使用以下命令安装 Go 的协议编译器插件：

     ```sh
     $ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
     $ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
     ```

  2. Update your `PATH` so that the `protoc` compiler can find the plugins: 更新您的 `PATH`，以便 `protoc` 编译器可以找到插件：

     ```sh
     $ export PATH="$PATH:$(go env GOPATH)/bin"
     ```

### 获取示例代码 Get the example code

The example code is part of the [grpc-go](https://github.com/grpc/grpc-go) repo.

​	示例代码位于 [grpc-go](https://github.com/grpc/grpc-go) 仓库中。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-go/archive/v1.68.0.zip) and unzip it, or clone the repo: [将仓库下载为 zip 文件](https://github.com/grpc/grpc-go/archive/v1.68.0.zip) 并解压，或克隆仓库：

   ```sh
   $ git clone -b v1.68.0 --depth 1 https://github.com/grpc/grpc-go
   ```

2. Change to the quick start example directory: 进入快速入门示例目录：

   ```sh
   $ cd grpc-go/examples/helloworld
   ```

### 运行示例 Run the example

From the `examples/helloworld` directory:

​	在 `examples/helloworld` 目录下：

1. Compile and execute the server code: 编译并执行服务器代码：

   ```sh
   $ go run greeter_server/main.go
   ```

2. From a different terminal, compile and execute the client code to see the client output: 在另一个终端中，编译并执行客户端代码以查看输出：

   ```sh
   $ go run greeter_client/main.go
   Greeting: Hello world
   ```

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚运行了一个使用 gRPC 的客户端-服务器应用程序。

### 更新 gRPC 服务 Update the gRPC service

In this section you’ll update the application with an extra server method. The gRPC service is defined using [protocol buffers](https://developers.google.com/protocol-buffers). To learn more about how to define a service in a `.proto` file see [Basics tutorial]({{< ref "/docs/Languages/Go/Basicstutorial" >}}). For now, all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

​	在本节中，您将为应用程序添加一个额外的服务器方法。gRPC 服务通过 [协议缓冲区](https://developers.google.com/protocol-buffers) 定义。有关如何在 `.proto` 文件中定义服务的更多信息，请参阅 [基础教程]({{< ref "/docs/Languages/Go/Basicstutorial" >}})。目前您只需知道，服务器和客户端存根都有一个 `SayHello()` RPC 方法，该方法从客户端接收一个 `HelloRequest` 参数并返回一个 `HelloReply`，定义如下：

```protobuf
// The greeting service definition.
// 问候服务定义。
service Greeter {
  // Sends a greeting
  // 发送问候
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
// 请求消息，包含用户姓名。
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
// 响应消息，包含问候内容。
message HelloReply {
  string message = 1;
}
```

Open `helloworld/helloworld.proto` and add a new `SayHelloAgain()` method, with the same request and response types:

​	打开 `helloworld/helloworld.proto` 文件，添加一个新方法 `SayHelloAgain()`，使用相同的请求和响应类型：

```protobuf
// The greeting service definition.
// 问候服务定义。
service Greeter {
  // Sends a greeting
  // 发送问候
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // Sends another greeting
  // 发送另一个问候
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
// 请求消息，包含用户姓名。
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
// 响应消息，包含问候内容。
message HelloReply {
  string message = 1;
}
```

Remember to save the file!

​	记得保存文件！

### 重新生成 gRPC 代码 Regenerate gRPC code

Before you can use the new service method, you need to recompile the updated `.proto` file.

​	在使用新服务方法之前，您需要重新编译更新后的 `.proto` 文件。

While still in the `examples/helloworld` directory, run the following command:

​	仍然在 `examples/helloworld` 目录中，运行以下命令：

```sh
$ protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    helloworld/helloworld.proto
```

This will regenerate the `helloworld/helloworld.pb.go` and `helloworld/helloworld_grpc.pb.go` files, which contain:

​	此操作将重新生成 `helloworld/helloworld.pb.go` 和 `helloworld/helloworld_grpc.pb.go` 文件，包含以下内容：

- Code for populating, serializing, and retrieving `HelloRequest` and `HelloReply` message types.
  - 填充、序列化和检索 `HelloRequest` 和 `HelloReply` 消息类型的代码。

- Generated client and server code.
  - 生成的客户端和服务器代码。


### 更新并运行应用程序 Update and run the application

You have regenerated server and client code, but you still need to implement and call the new method in the human-written parts of the example application.

​	您已经重新生成了服务器和客户端代码，但仍需在示例应用程序的人为编写部分中实现并调用新方法。

#### 更新服务器 Update the server

Open `greeter_server/main.go` and add the following function to it:

​	打开 `greeter_server/main.go` 文件，添加以下函数：

```go
func (s *server) SayHelloAgain(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
        return &pb.HelloReply{Message: "Hello again " + in.GetName()}, nil
}
```

#### 更新客户端 Update the client

Open `greeter_client/main.go` to add the following code to the end of the `main()` function body:

​	打开 `greeter_client/main.go` 文件，在 `main()` 函数体的末尾添加以下代码：

```go
r, err = c.SayHelloAgain(ctx, &pb.HelloRequest{Name: *name})
if err != nil {
        log.Fatalf("could not greet: %v", err)
}
log.Printf("Greeting: %s", r.GetMessage())
```

Remember to save your changes.

​	记得保存更改。

#### Run!

Run the client and server like you did before. Execute the following commands from the `examples/helloworld` directory:

​	像之前一样运行客户端和服务器。从 `examples/helloworld` 目录执行以下命令：

1. Run the server:

   ```sh
   $ go run greeter_server/main.go
   ```

2. From another terminal, run the client. This time, add a name as a command-line argument: 在另一个终端中运行客户端。这次，添加一个名字作为命令行参数：

   ```sh
   $ go run greeter_client/main.go --name=Alice
   ```

   You’ll see the following output:

   ​	您将看到以下输出：
   
   ```sh
   Greeting: Hello Alice
   Greeting: Hello again Alice
   ```

### What’s next

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/docs/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/docs/WhatisgRPC/Coreconcepts" >}}).
  - 了解 gRPC 的工作原理，参阅 [gRPC 简介]({{< ref "/docs/WhatisgRPC/Introduction" >}}) 和 [核心概念]({{< ref "/docs/WhatisgRPC/Coreconcepts" >}})。
- Work through the [Basics tutorial]({{< ref "/docs/Languages/Go/Basicstutorial" >}}).
  - 练习 [基础教程]({{< ref "/docs/Languages/Go/Basicstutorial" >}})。
- Explore the [API reference]({{< ref "/docs/Languages/Go/API" >}}).
  - 探索 [API 参考]({{< ref "/docs/Languages/Go/API" >}})。
