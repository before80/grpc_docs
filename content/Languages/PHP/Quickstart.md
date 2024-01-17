+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/php/quickstart/](https://grpc.io/docs/languages/php/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC in PHP with a simple working example.

​	本指南通过一个简单的示例帮助您开始在 PHP 中使用 gRPC。



### Prerequisites 先决条件

- PHP 7.0 or higher, PECL, Composer
  PHP 7.0 或更高版本、PECL、Composer
- grpc extension, protocol buffers compiler: for installation instructions, see the [gRPC PHP readme](https://github.com/grpc/grpc/blob/v1.60.0/src/php/README.md).
  grpc 扩展、协议缓冲区编译器：有关安装说明，请参阅 gRPC PHP 自述文件。

### Get the example code 获取示例代码

The example code is part of the [grpc](https://github.com/grpc/grpc) repo.

​	示例代码是 grpc 代码库的一部分。

#### Note 注意

You can only create gRPC clients in PHP. Use [another language](https://grpc.io/docs/languages/) to create a gRPC server.
您只能在 PHP 中创建 gRPC 客户端。使用其他语言创建 gRPC 服务器。

1. Clone the [grpc](https://github.com/grpc/grpc) repo and its submodules:

   ​	克隆 grpc 代码库及其子模块：

   ```sh
   $ git clone --recurse-submodules -b v1.60.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc
   ```

2. Change to the quick start example directory:

   ​	转到快速入门示例目录：

   ```sh
   $ cd grpc/examples/php
   ```

3. Install the `grpc` composer package:

   ​	安装 `grpc` composer 包：

   ```sh
   $ ./greeter_proto_gen.sh
   $ composer install
   ```

### Run the example 运行示例

1. Launch the quick start server: for example, follow the instructions given in the [Quick start for Node]({{< ref "/Languages/Node/Quickstart" >}}).

   ​	启动快速入门服务器：例如，按照 Node 的快速入门中给出的说明进行操作。

2. From the `examples/php` directory, run the PHP client:

   ​	从 `examples/php` 目录运行 PHP 客户端：

   ```sh
   $ ./run_greeter_client.sh
   ```

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

### Update the gRPC service 更新 gRPC 服务

Now let’s look at how to update the application with an extra method on the server for the client to call. Our gRPC service is defined using protocol buffers; you can find out lots more about how to define a service in a `.proto` file in [Basics tutorial]({{< ref "/Languages/PHP/Basicstutorial" >}}). For now all you need to know is that both the server and the client “stub” have a `SayHello` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloResponse` from the server, and that this method is defined like this:

​	现在，我们来看看如何使用服务器上的额外方法更新应用程序以供客户端调用。我们的 gRPC 服务是使用协议缓冲区定义的；您可以在基本教程中找到有关如何在 `.proto` 文件中定义服务的更多信息。现在，您只需要知道服务器和客户端“存根”都具有 `SayHello` RPC 方法，该方法从客户端获取 `HelloRequest` 参数并从服务器返回 `HelloResponse` ，并且此方法的定义如下：

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

Let’s update this so that the `Greeter` service has two methods. Edit `examples/protos/helloworld.proto` and update it with a new `SayHelloAgain` method, with the same request and response types:

​	让我们更新此内容，以便 `Greeter` 服务具有两种方法。编辑 `examples/protos/helloworld.proto` 并使用新的 `SayHelloAgain` 方法对其进行更新，该方法具有相同的请求和响应类型：

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

Next we need to update the gRPC code used by our application to use the new service definition. From the `grpc` root directory:

​	接下来，我们需要更新应用程序使用的 gRPC 代码，以使用新的服务定义。从 `grpc` 根目录：

```sh
$ protoc --proto_path=examples/protos \
  --php_out=examples/php \
  --grpc_out=examples/php \
  --plugin=protoc-gen-grpc=bins/opt/grpc_php_plugin \
  ./examples/protos/helloworld.proto
```

or running the helper script under the `grpc/example/php` directory if you build grpc-php-plugin by source:

​	或在 `grpc/example/php` 目录下运行帮助程序脚本（如果您通过源代码构建 grpc-php-plugin）：

```sh
$ ./greeter_proto_gen.sh
```

This regenerates the protobuf files, which contain our generated client classes, as well as classes for populating, serializing, and retrieving our request and response types.

​	这会重新生成 protobuf 文件，其中包含我们生成的客户端类，以及用于填充、序列化和检索我们的请求和响应类型的类。

### Update and run the application 更新并运行应用程序

We now have new generated client code, but we still need to implement and call the new method in the human-written parts of our example application.

​	我们现在有了新的生成的客户端代码，但我们仍然需要在示例应用程序的手写部分中实现和调用新方法。

#### Update the server 从示例的根目录

In the same directory, open `greeter_server.js`. Implement the new method like this:

​	在同一目录中，打开 `greeter_server.js` 。像这样实现新方法：

```js
function sayHello(call, callback) {
  callback(null, {message: 'Hello ' + call.request.name});
}

function sayHelloAgain(call, callback) {
  callback(null, {message: 'Hello again, ' + call.request.name});
}

function main() {
  var server = new grpc.Server();
  server.addProtoService(hello_proto.Greeter.service,
                         {sayHello: sayHello, sayHelloAgain: sayHelloAgain});
  server.bind('0.0.0.0:50051', grpc.ServerCredentials.createInsecure());
  server.start();
}
...
```

#### Update the client 更新客户端

In the same directory, open `greeter_client.php`. Call the new method like this:

​	在同一目录中，打开 `greeter_client.php` 。像这样调用新方法：

```php
$request = new Helloworld\HelloRequest();
$request->setName($name);
list($reply, $status) = $client->SayHello($request)->wait();
$message = $reply->getMessage();
list($reply, $status) = $client->SayHelloAgain($request)->wait();
$message = $reply->getMessage();
```

#### Run! 运行！

Just like we did before, from the `grpc-node/examples/helloworld/dynamic_codegen` directory:

​	就像我们之前从 `grpc-node/examples/helloworld/dynamic_codegen` 目录中所做的那样：

1. Run the server: 
   ​	运行服务器：

   ```sh
   $ node greeter_server.js
   ```

2. From another terminal, from the `examples/php` directory, run the client:

   ​	从另一个终端，从 `examples/php` 目录运行客户端：

   ```sh
   $ ./run_greeter_client.sh
   ```

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Languages/PHP/Basicstutorial" >}}).
  完成基础知识教程。
- Explore the [API reference]({{< ref "/Languages/PHP/API" >}}).
  探索 API 参考。
