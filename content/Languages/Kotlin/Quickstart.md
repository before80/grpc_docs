+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/kotlin/quickstart/](https://grpc.io/docs/languages/kotlin/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC in Kotlin with a simple working example.

​	本指南通过一个简单的示例帮助您开始在 Kotlin 中使用 gRPC。



### Prerequisites 先决条件

- [Kotlin](https://kotlinlang.org/) version 1.3 or higher
  Kotlin 版本 1.3 或更高
- [JDK](https://jdk.java.net/) version 7 or higher
  JDK 版本 7 或更高

### Get the example code 获取示例代码

The example code is part of the [grpc-kotlin](https://github.com/grpc/grpc-kotlin) repo.

​	示例代码是 grpc-kotlin 代码库的一部分。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-kotlin/archive/master.zip) and unzip it, or clone the repo:

   ​	以 zip 文件形式下载代码库并解压缩，或克隆代码库：

   ```sh
   $ git clone --depth 1 https://github.com/grpc/grpc-kotlin
   ```

2. Change to the examples directory:

   ​	更改到 examples 目录：

   ```sh
   $ cd grpc-kotlin/examples
   ```

### Run the example 运行示例

From the `examples` directory:

​	从 `examples` 目录：

1. Compile the client and server

   ​	编译客户端和服务器

   ```sh
   $ ./gradlew installDist
   ```

2. Run the server: 
   ​	运行服务器：

   ```sh
   $ ./server/build/install/server/bin/hello-world-server
   Server started, listening on 50051
   ```

3. From another terminal, run the client:

   ​	从另一个终端运行客户端：

   ```sh
   $ ./client/build/install/client/bin/hello-world-client
   Received: Hello world
   ```

Congratulations! You’ve just run a client-server app with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

### Update the gRPC service 更新 gRPC 服务

In this section, you’ll update the app with an extra server method. The app’s gRPC service, named `Greeter`, is defined using [protocol buffers](https://developers.google.com/protocol-buffers). To learn more about how to define a service in a `.proto` file, see [Basics tutorial]({{< ref "/Languages/Kotlin/Basicstutorial" >}}). For now, all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

​	在本部分中，您将使用一个额外的服务器方法更新应用程序。该应用程序的 gRPC 服务名为 `Greeter` ，是使用 Protocol Buffers 定义的。要详细了解如何在 `.proto` 文件中定义服务，请参阅基础知识教程。现在，您只需要知道服务器和客户端存根都具有 `SayHello()` RPC 方法，该方法从客户端获取 `HelloRequest` 参数并从服务器返回 `HelloReply` ，并且该方法的定义如下：

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

Open `helloworld/hello_world.proto` from the [protos/src/main/proto/io/grpc/examples](https://github.com/grpc/grpc-kotlin/tree/master/examples/protos/src/main/proto/io/grpc/examples) folder, and add a new `SayHelloAgain()` method, with the same request and response types:

​	从 protos/src/main/proto/io/grpc/examples 文件夹中打开 `helloworld/hello_world.proto` ，并添加一个新的 `SayHelloAgain()` 方法，具有相同的请求和响应类型：

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

### Update the app 更新应用

When you build the example, the build process regenerates `HelloWorldProtoGrpcKt.kt`, which contains the generated gRPC client and server classes. This also regenerates classes for populating, serializing, and retrieving our request and response types.

​	在您构建示例时，构建过程会重新生成 `HelloWorldProtoGrpcKt.kt` ，其中包含生成的 gRPC 客户端和服务器类。这还会重新生成用于填充、序列化和检索我们的请求和响应类型的类。

However, you still need to implement and call the new method in the hand-written parts of the example app.

​	但是，您仍然需要在示例应用的手写部分中实现并调用新方法。

#### Update the server 从示例的根目录

Open `helloworld/HelloWorldServer.kt` from the [server/src/main/kotlin/io/grpc/examples](https://github.com/grpc/grpc-kotlin/tree/master/examples/server/src/main/kotlin/io/grpc/examples) folder. Implement the new method like this:

​	从 server/src/main/kotlin/io/grpc/examples 文件夹中打开 `helloworld/HelloWorldServer.kt` 。像这样实现新方法：

```kotlin
private class HelloWorldService : GreeterGrpcKt.GreeterCoroutineImplBase() {
  override suspend fun sayHello(request: HelloRequest) = helloReply {
    message = "Hello ${request.name}"
  }

  override suspend fun sayHelloAgain(request: HelloRequest) = helloReply {
    message = "Hello again ${request.name}"
  }
}
```

#### Update the client 更新客户端

Open `helloworld/HelloWorldClient.kt` from the [client/src/main/kotlin/io/grpc/examples](https://github.com/grpc/grpc-kotlin/tree/master/examples/client/src/main/kotlin/io/grpc/examples) folder. Call the new method like this:

​	从 client/src/main/kotlin/io/grpc/examples 文件夹中打开 `helloworld/HelloWorldClient.kt` 。像这样调用新方法：

```kotlin
class HelloWorldClient(
    private val channel: ManagedChannel
) : Closeable {
  private val stub: GreeterCoroutineStub = GreeterCoroutineStub(channel)

  suspend fun greet(name: String) {
    val request = helloRequest { this.name = name }
    val response = stub.sayHello(request)
    println("Received: ${response.message}")
    val againResponse = stub.sayHelloAgain(request)
    println("Received: ${againResponse.message}")
  }

  override fun close() {
    channel.shutdown().awaitTermination(5, TimeUnit.SECONDS)
  }
}
```

### Run the updated app 运行更新后的应用

Run the client and server like you did before. Execute the following commands from the `examples` directory:

​	像以前一样运行客户端和服务器。从 `examples` 目录执行以下命令：

1. Compile the client and server:

   ​	编译客户端和服务器：

   ```sh
   $ ./gradlew installDist
   ```

2. Run the server: 
   ​	运行服务器：

   ```sh
   $ ./server/build/install/server/bin/hello-world-server
   Server started, listening on 50051
   ```

3. From another terminal, run the client. This time, add a name as a command-line argument:

   ​	从另一个终端运行客户端。这次，添加一个名称作为命令行参数：

   ```sh
   $ ./client/build/install/client/bin/hello-world-client Alice
   Received: Hello Alice
   Received: Hello again Alice
   ```

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Languages/Kotlin/Basicstutorial" >}}).
  完成基础知识教程。
- Explore the [API reference]({{< ref "/Languages/Kotlin/API" >}}).
  探索 API 参考。
