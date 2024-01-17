+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/node/quickstart/](https://grpc.io/docs/languages/node/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC in Node with a simple working example.

​	本指南通过一个简单的示例帮助您开始在 Node 中使用 gRPC。



### Prerequisites 先决条件

- Node version 8.13.0 or higher
  Node 版本 8.13.0 或更高

### Download the example 下载示例

You’ll need a local copy of the example code to work through this quick start. Download the example code from our GitHub repository (the following command clones the entire repository, but you just need the examples for this quick start and other tutorials):

​	您需要示例代码的本地副本才能完成此快速入门。从我们的 GitHub 存储库下载示例代码（以下命令克隆整个存储库，但您只需要此快速入门和其他教程的示例）：

```sh
# Clone the repository to get the example code
$ git clone -b @grpc/grpc-js@1.9.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc-node
# Navigate to the node example
$ cd grpc-node/examples
# Install the example's dependencies
$ npm install
# Navigate to the dynamic codegen "hello, world" Node example:
$ cd helloworld/dynamic_codegen
```

### Run a gRPC application 运行 gRPC 应用程序

From the `examples/helloworld/dynamic_codegen` directory:

​	从 `examples/helloworld/dynamic_codegen` 目录：

1. Run the server: 
   ​	运行服务器：

   ```sh
   $ node greeter_server.js
   ```

2. From another terminal, run the client:

   ​	从另一个终端运行客户端：

   ```sh
   $ node greeter_client.js
   ```

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

### Update the gRPC service 更新 gRPC 服务

Now let’s look at how to update the application with an extra method on the server for the client to call. Our gRPC service is defined using protocol buffers; you can find out lots more about how to define a service in a `.proto` file in [Basics tutorial]({{< ref "/Languages/Node/Basicstutorial" >}}). For now all you need to know is that both the server and the client “stub” have a `SayHello` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that this method is defined like this:

​	现在，我们来看看如何使用服务器上的额外方法更新应用程序以供客户端调用。我们的 gRPC 服务是使用协议缓冲区定义的；您可以在基本教程中找到有关如何在 `.proto` 文件中定义服务的更多信息。现在，您只需要知道服务器和客户端“存根”都具有 `SayHello` RPC 方法，该方法从客户端获取 `HelloRequest` 参数并从服务器返回 `HelloReply` ，并且此方法的定义如下：

```proto
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

```proto
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

### Update and run the application 更新并运行应用程序

We now have a new service definition, but we still need to implement and call the new method in the human-written parts of our example application.

​	我们现在有了新的服务定义，但我们仍然需要在示例应用程序的人工编写部分中实现和调用新方法。

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
  server.addService(hello_proto.Greeter.service,
                         {sayHello: sayHello, sayHelloAgain: sayHelloAgain});
  server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {
    server.start();
  });
}
```

#### Update the client 更新客户端

In the same directory, open `greeter_client.js`. Call the new method like this:

​	在同一目录中，打开 `greeter_client.js` 。像这样调用新方法：

```js
function main() {
  var client = new hello_proto.Greeter('localhost:50051',
                                       grpc.credentials.createInsecure());
  client.sayHello({name: 'you'}, function(err, response) {
    console.log('Greeting:', response.message);
  });
  client.sayHelloAgain({name: 'you'}, function(err, response) {
    console.log('Greeting:', response.message);
  });
}
```

#### Run! 运行！

Just like we did before, from the `examples/helloworld/dynamic_codegen` directory:

​	就像我们之前从 `examples/helloworld/dynamic_codegen` 目录中所做的那样：

1. Run the server: 
   ​	运行服务器：

   ```sh
   $ node greeter_server.js
   ```

2. From another terminal, run the client:

   ​	从另一个终端运行客户端：

   ```sh
   $ node greeter_client.js
   ```

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Languages/Node/Basicstutorial" >}}).
  完成基础知识教程。
- Explore the [API reference]({{< ref "/Languages/Node/API" >}}).
  探索 API 参考。
- We have more than one grpc implementation for Node. For the pros and cons of each package, see this [package feature comparison](https://github.com/grpc/grpc-node/blob/master/PACKAGE-COMPARISON.md).
  我们为 Node 提供了多个 grpc 实现。有关每个软件包的优缺点，请参阅此软件包功能比较。
