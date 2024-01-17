+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/dart/quickstart/](https://grpc.io/docs/languages/dart/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC in Dart with a simple working example.

​	本指南通过一个简单的示例帮助您开始在 Dart 中使用 gRPC。



### Prerequisites 先决条件

- **[Dart](https://dart.dev/)** version 2.12 or higher, through the Dart or [Flutter](https://flutter.dev/) SDKs

  ​	Dart 版本 2.12 或更高版本，通过 Dart 或 Flutter SDK

  For installation instructions, see [Install Dart](https://dart.dev/install) or [Install Flutter](https://flutter.dev/docs/get-started/install).

  ​	有关安装说明，请参阅安装 Dart 或安装 Flutter。

- **[Protocol buffer](https://developers.google.com/protocol-buffers) compiler**, `protoc`, [version 3](https://protobuf.dev/programming-guides/proto3)

  ​	协议缓冲区编译器， `protoc` ，版本 3

  For installation instructions, see [Protocol Buffer Compiler Installation](https://grpc.io/docs/protoc-installation/).

  ​	有关安装说明，请参阅协议缓冲区编译器安装。

- **Dart plugin** for the protocol compiler:

  ​	协议编译器的 Dart 插件：

  1. Install the protocol compiler plugin for Dart (`protoc-gen-dart`) using the following command:

     ​	使用以下命令安装适用于 Dart 的协议编译器插件 ( `protoc-gen-dart` )：

     ```sh
     $ dart pub global activate protoc_plugin
     ```

  2. Update your `PATH` so that the `protoc` compiler can find the plugin:

     ​	更新您的 `PATH` ，以便 `protoc` 编译器可以找到该插件：

     ```sh
     $ export PATH="$PATH:$HOME/.pub-cache/bin"
     ```

#### Note 注意

Dart gRPC supports the Flutter and Server platforms.
Dart gRPC 支持 Flutter 和 Server 平台。

### Get the example code 获取示例代码

The example code is part of the [grpc-dart](https://github.com/grpc/grpc-dart) repo.

​	示例代码是 grpc-dart 代码库的一部分。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-dart/archive/master.zip) and unzip it, or clone the repo:

   ​	以 zip 文件形式下载代码库并解压缩，或克隆代码库：

   ```sh
   $ git clone https://github.com/grpc/grpc-dart
   ```

2. Change to the quick start example directory:

   ​	转到快速入门示例目录：

   ```sh
   $ cd grpc-dart/example/helloworld
   ```

### Run the example 运行示例

From the `example/helloworld` directory:

​	从 `example/helloworld` 目录：

1. Download package dependencies:

   ​	下载软件包依赖项：

   ```sh
   $ dart pub get
   ```

2. Run the server: 
   ​	运行服务器：

   ```sh
   $ dart bin/server.dart
   ```

3. From another terminal, run the client:

   ​	从另一个终端运行客户端：

   ```sh
   $ dart bin/client.dart
   Greeter client received: Hello, world!
   ```

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

### Update the app 更新应用

In this section you’ll update the app to make use of an extra server method. The gRPC service is defined using [protocol buffers](https://developers.google.com/protocol-buffers). To learn more about how to define a service in a `.proto` file, see [Basics tutorial]({{< ref "/Languages/Dart/Basicstutorial" >}}). For now, all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

​	在本节中，您将更新应用以利用额外的服务器方法。gRPC 服务使用协议缓冲区定义。要详细了解如何在 `.proto` 文件中定义服务，请参阅基础教程。现在，您只需要知道服务器和客户端存根都具有 `SayHello()` RPC 方法，该方法采用来自客户端的 `HelloRequest` 参数并从服务器返回 `HelloReply` ，并且该方法的定义如下：

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

#### Update the gRPC service 更新 gRPC 服务

Open `protos/helloworld.proto` and add a new `SayHelloAgain()` method, with the same request and response types:

​	打开 `protos/helloworld.proto` 并添加一个新的 `SayHelloAgain()` 方法，具有相同的请求和响应类型：

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

#### Regenerate gRPC code 重新生成 gRPC 代码

Before you can use the new service method, you need to recompile the updated proto file. From the `example/helloworld` directory, run the following command:

​	在使用新的服务方法之前，您需要重新编译更新后的 proto 文件。从 `example/helloworld` 目录运行以下命令：

```sh
$ protoc --dart_out=grpc:lib/src/generated -Iprotos protos/helloworld.proto
```

You’ll find the regenerated request and response classes, and client and server classes in the `lib/src/generated` directory.

​	您将在 `lib/src/generated` 目录中找到重新生成请求和响应类，以及客户端和服务器类。

Now implement and call the new RPC in the server and client code, respectively.

​	现在分别在服务器和客户端代码中实现并调用新的 RPC。

#### Update the server 从示例的根目录

Open `bin/server.dart` and add the following `sayHelloAgain()` method to the `GreeterService` class:

​	打开 `bin/server.dart` ，并将以下 `sayHelloAgain()` 方法添加到 `GreeterService` 类中：

```dart
class GreeterService extends GreeterServiceBase {
  @override
  Future<HelloReply> sayHello(ServiceCall call, HelloRequest request) async {
    return HelloReply()..message = 'Hello, ${request.name}!';
  }

  @override
  Future<HelloReply> sayHelloAgain(ServiceCall call, HelloRequest request) async {
    return HelloReply()..message = 'Hello again, ${request.name}!';
  }
}
```

#### Update the client 更新客户端

Add a call to `sayHelloAgain()` in `bin/client.dart` like this:

​	在 `bin/client.dart` 中添加对 `sayHelloAgain()` 的调用，如下所示：

```dart
Future<void> main(List<String> args) async {
  final channel = ClientChannel(
    'localhost',
    port: 50051,
    options: const ChannelOptions(credentials: ChannelCredentials.insecure()),
  );
  final stub = GreeterClient(channel);

  final name = args.isNotEmpty ? args[0] : 'world';

  try {
    var response = await stub.sayHello(HelloRequest()..name = name);
    print('Greeter client received: ${response.message}');
    response = await stub.sayHelloAgain(HelloRequest()..name = name);
    print('Greeter client received: ${response.message}');
  } catch (e) {
    print('Caught error: $e');
  }
  await channel.shutdown();
}
```

### Run the updated app 运行更新后的应用

Run the client and server like you did before. Execute the following commands from the `example/helloworld` directory:

​	像以前一样运行客户端和服务器。从 `example/helloworld` 目录执行以下命令：

1. Run the server: 
   ​	运行服务器：

   ```sh
   $ dart bin/server.dart
   ```

2. From another terminal, run the client. This time, add a name as a command-line argument:

   ​	从另一个终端运行客户端。这次，添加一个名称作为命令行参数：

   ```sh
   $ dart bin/client.dart Alice
   ```

   You’ll see the following output:

   ​	您将看到以下输出：

   ```sh
   Greeter client received: Hello, Alice!
   Greeter client received: Hello again, Alice!
   ```

### Contributing 贡献

If you experience problems with Dart gRPC or have a feature request, [create an issue](https://github.com/grpc/grpc-dart/issues/new) over the [grpc-dart](https://github.com/grpc/grpc-dart) repo.

​	如果您遇到 Dart gRPC 问题或有功能请求，请在 grpc-dart 代码库中创建问题。

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Languages/Dart/Basicstutorial" >}}).
  完成基础知识教程。
- Explore the [API reference]({{< ref "/Languages/Dart/API" >}}).
  探索 API 参考。
