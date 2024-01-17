+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/go/quickstart/](https://grpc.io/docs/languages/go/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC in Go with a simple working example.

​	本指南通过一个简单的示例帮助您开始在 Go 中使用 gRPC。



### Prerequisites 先决条件

- **[Go](https://golang.org/)**, any one of the **three latest major** [releases of Go](https://golang.org/doc/devel/release.html).

  ​	Go，三个最新主要版本中的任何一个。

  For installation instructions, see Go’s [Getting Started](https://golang.org/doc/install) guide.

  ​	有关安装说明，请参阅 Go 的入门指南。

- **[Protocol buffer](https://developers.google.com/protocol-buffers) compiler**, `protoc`, [version 3](https://protobuf.dev/programming-guides/proto3).

  ​	协议缓冲区编译器， `protoc` ，版本 3。

  For installation instructions, see [Protocol Buffer Compiler Installation](https://grpc.io/docs/protoc-installation/).

  ​	有关安装说明，请参阅协议缓冲区编译器安装。

- **Go plugins** for the protocol compiler:

  ​	适用于协议编译器的 Go 插件：

  1. Install the protocol compiler plugins for Go using the following commands:

     ​	使用以下命令安装适用于 Go 的协议编译器插件：

     ```sh
     $ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
     $ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
     ```

  2. Update your `PATH` so that the `protoc` compiler can find the plugins:

     ​	更新您的 `PATH` ，以便 `protoc` 编译器可以找到插件：

     ```sh
     $ export PATH="$PATH:$(go env GOPATH)/bin"
     ```

### Get the example code 获取示例代码

The example code is part of the [grpc-go](https://github.com/grpc/grpc-go) repo.

​	示例代码是 grpc-go 存储库的一部分。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-go/archive/v1.60.1.zip) and unzip it, or clone the repo:

   ​	以 zip 文件形式下载代码库并解压缩，或克隆代码库：

   ```sh
   $ git clone -b v1.60.1 --depth 1 https://github.com/grpc/grpc-go
   ```

2. Change to the quick start example directory:

   ​	转到快速入门示例目录：

   ```sh
   $ cd grpc-go/examples/helloworld
   ```

### Run the example 运行示例

From the `examples/helloworld` directory:

​	从 `examples/helloworld` 目录：

1. Compile and execute the server code:

   ​	编译并执行服务器代码：

   ```sh
   $ go run greeter_server/main.go
   ```

2. From a different terminal, compile and execute the client code to see the client output:

   ​	从另一个终端编译并执行客户端代码以查看客户端输出：

   ```sh
   $ go run greeter_client/main.go
   Greeting: Hello world
   ```

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

### Update the gRPC service 更新 gRPC 服务

In this section you’ll update the application with an extra server method. The gRPC service is defined using [protocol buffers](https://developers.google.com/protocol-buffers). To learn more about how to define a service in a `.proto` file see [Basics tutorial]({{< ref "/Languages/Go/Basicstutorial" >}}). For now, all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

​	在本部分中，您将使用额外的服务器方法更新应用程序。gRPC 服务使用协议缓冲区定义。要详细了解如何在 `.proto` 文件中定义服务，请参阅基本教程。现在，您只需要知道服务器和客户端存根都具有 `SayHello()` RPC 方法，该方法采用来自客户端的 `HelloRequest` 参数并从服务器返回 `HelloReply` ，并且该方法的定义如下：

```protobuf
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

Open `helloworld/helloworld.proto` and add a new `SayHelloAgain()` method, with the same request and response types:

​	打开 `helloworld/helloworld.proto` 并添加一个新的 `SayHelloAgain()` 方法，具有相同的请求和响应类型：

```protobuf
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // Sends another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

Remember to save the file!

​	请记住保存文件！

### Regenerate gRPC code 重新生成 gRPC 代码

Before you can use the new service method, you need to recompile the updated `.proto` file.

​	在您能够使用新的服务方法之前，您需要重新编译更新的 `.proto` 文件。

While still in the `examples/helloworld` directory, run the following command:

​	仍在 `examples/helloworld` 目录中时，运行以下命令：

```sh
$ protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    helloworld/helloworld.proto
```

This will regenerate the `helloworld/helloworld.pb.go` and `helloworld/helloworld_grpc.pb.go` files, which contain:

​	这将重新生成 `helloworld/helloworld.pb.go` 和 `helloworld/helloworld_grpc.pb.go` 文件，其中包含：

- Code for populating, serializing, and retrieving `HelloRequest` and `HelloReply` message types.
  用于填充、序列化和检索 `HelloRequest` 和 `HelloReply` 消息类型的代码。
- Generated client and server code.
  生成的客户端和服务器代码。

### Update and run the application 更新并运行应用程序

You have regenerated server and client code, but you still need to implement and call the new method in the human-written parts of the example application.

​	您已重新生成服务器和客户端代码，但您仍需要在示例应用程序的手写部分中实现和调用新方法。

#### Update the server 从示例的根目录

Open `greeter_server/main.go` and add the following function to it:

​	打开 `greeter_server/main.go` 并向其中添加以下函数：

```go
func (s *server) SayHelloAgain(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
        return &pb.HelloReply{Message: "Hello again " + in.GetName()}, nil
}
```

#### Update the client 更新客户端

Open `greeter_client/main.go` to add the following code to the end of the `main()` function body:

​	打开 `greeter_client/main.go` 以将以下代码添加到 `main()` 函数体的末尾：

```go
r, err = c.SayHelloAgain(ctx, &pb.HelloRequest{Name: *name})
if err != nil {
        log.Fatalf("could not greet: %v", err)
}
log.Printf("Greeting: %s", r.GetMessage())
```

Remember to save your changes.

​	请记住保存您的更改。

#### Run! 运行！

Run the client and server like you did before. Execute the following commands from the `examples/helloworld` directory:

​	像以前一样运行客户端和服务器。从 `examples/helloworld` 目录执行以下命令：

1. Run the server: 
   ​	运行服务器：

   ```sh
   $ go run greeter_server/main.go
   ```

2. From another terminal, run the client. This time, add a name as a command-line argument:

   ​	从另一个终端运行客户端。这次，添加一个名称作为命令行参数：

   ```sh
   $ go run greeter_client/main.go --name=Alice
   ```

   You’ll see the following output:

   ​	您将看到以下输出：

   ```sh
   Greeting: Hello Alice
   Greeting: Hello again Alice
   ```

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Languages/Go/Basicstutorial" >}}).
  完成基础知识教程。
- Explore the [API reference]({{< ref "/Languages/Go/API" >}}).
  探索 API 参考。
