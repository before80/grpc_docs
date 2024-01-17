+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/java/quickstart/](https://grpc.io/docs/languages/java/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC in Java with a simple working example.

​	本指南通过一个简单的示例帮助您开始在 Java 中使用 gRPC。



### Prerequisites 先决条件

- [JDK](https://jdk.java.net/) version 7 or higher
  JDK 版本 7 或更高

### Get the example code 获取示例代码

The example code is part of the [grpc-java](https://github.com/grpc/grpc-java) repo.

​	示例代码是 grpc-java 代码库的一部分。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-java/archive/v1.61.0.zip) and unzip it, or clone the repo:

   ​	以 zip 文件形式下载代码库并解压缩，或克隆代码库：

   ```sh
   $ git clone -b v1.61.0 --depth 1 https://github.com/grpc/grpc-java
   ```

2. Change to the examples directory:

   ​	更改到 examples 目录：

   ```sh
   $ cd grpc-java/examples
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
   $ ./build/install/examples/bin/hello-world-server
   INFO: Server started, listening on 50051
   ```

3. From another terminal, run the client:

   ​	从另一个终端运行客户端：

   ```sh
   $ ./build/install/examples/bin/hello-world-client
   INFO: Will try to greet world ...
   INFO: Greeting: Hello world
   ```

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

#### Note 注意

We’ve omitted timestamps from the client and server trace output shown in this page.
我们已从本页中显示的客户端和服务器跟踪输出中省略了时间戳。

### Update the gRPC service 更新 gRPC 服务

In this section you’ll update the application by adding an extra server method. The gRPC service is defined using [protocol buffers](https://developers.google.com/protocol-buffers). To learn more about how to define a service in a `.proto` file see [Basics tutorial]({{< ref "/Languages/Java/Basicstutorial" >}}). For now, all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

​	在本节中，您将通过添加一个额外的服务器方法来更新应用程序。gRPC 服务是使用协议缓冲区定义的。要详细了解如何在 `.proto` 文件中定义服务，请参阅基础知识教程。现在，您只需要知道服务器和客户端存根都具有 `SayHello()` RPC 方法，该方法采用来自客户端的 `HelloRequest` 参数，并从服务器返回 `HelloReply` ，并且该方法的定义如下：

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

Open `src/main/proto/helloworld.proto` and add a new `SayHelloAgain()` method with the same request and response types as `SayHello()`:

​	打开 `src/main/proto/helloworld.proto` 并添加一个新的 `SayHelloAgain()` 方法，其请求和响应类型与 `SayHello()` 相同：

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

When you build the example, the build process regenerates `GreeterGrpc.java`, which contains the generated gRPC client and server classes. This also regenerates classes for populating, serializing, and retrieving our request and response types.

​	在您构建示例时，构建过程会重新生成 `GreeterGrpc.java` ，其中包含生成的 gRPC 客户端和服务器类。这还会重新生成用于填充、序列化和检索我们的请求和响应类型的类。

However, you still need to implement and call the new method in the hand-written parts of the example app.

​	但是，您仍然需要在示例应用的手写部分中实现并调用新方法。

#### Update the server 从示例的根目录

In the same directory, open `src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java`. Implement the new method like this:

​	在同一目录中，打开 `src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java` 。像这样实现新方法：

```java
private class GreeterImpl extends GreeterGrpc.GreeterImplBase {

  @Override
  public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello " + req.getName()).build();
    responseObserver.onNext(reply);
    responseObserver.onCompleted();
  }

  @Override
  public void sayHelloAgain(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello again " + req.getName()).build();
    responseObserver.onNext(reply);
    responseObserver.onCompleted();
  }
}
```

#### Update the client 更新客户端

In the same directory, open `src/main/java/io/grpc/examples/helloworld/HelloWorldClient.java`. Call the new method like this:

​	在同一目录中，打开 `src/main/java/io/grpc/examples/helloworld/HelloWorldClient.java` 。像这样调用新方法：

```java
public void greet(String name) {
  logger.info("Will try to greet " + name + " ...");
  HelloRequest request = HelloRequest.newBuilder().setName(name).build();
  HelloReply response;
  try {
    response = blockingStub.sayHello(request);
  } catch (StatusRuntimeException e) {
    logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
    return;
  }
  logger.info("Greeting: " + response.getMessage());
  try {
    response = blockingStub.sayHelloAgain(request);
  } catch (StatusRuntimeException e) {
    logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
    return;
  }
  logger.info("Greeting: " + response.getMessage());
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
   $ ./build/install/examples/bin/hello-world-server
   INFO: Server started, listening on 50051
   ```

3. From another terminal, run the client:

   ​	从另一个终端运行客户端：

   ```sh
   $ ./build/install/examples/bin/hello-world-client
   INFO: Will try to greet world ...
   INFO: Greeting: Hello world
   INFO: Greeting: Hello again world
   ```

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Languages/Java/Basicstutorial" >}}).
  完成基础知识教程。
- Explore the [API reference]({{< ref "/Languages/Java/API" >}}).
  探索 API 参考。
