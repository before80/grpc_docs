+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/python/quickstart/](https://grpc.io/docs/languages/python/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC in Python with a simple working example.

​	本指南通过一个简单的示例帮助您开始使用 Python 中的 gRPC。



### Prerequisites 先决条件

- Python 3.7 or higher Python 3.7 或更高版本
- `pip` version 9.0.1 or higher
  `pip` 9.0.1 或更高版本

If necessary, upgrade your version of `pip`:

​	如有必要，请升级您的 `pip` 版本：

```sh
$ python -m pip install --upgrade pip
```

If you cannot upgrade `pip` due to a system-owned installation, you can run the example in a virtualenv:

​	如果您无法升级 `pip` ，因为它是系统所有安装，您可以在 virtualenv 中运行示例：

```sh
$ python -m pip install virtualenv
$ virtualenv venv
$ source venv/bin/activate
$ python -m pip install --upgrade pip
```

#### gRPC

Install gRPC: 
​	安装 gRPC：

```sh
$ python -m pip install grpcio
```

Or, to install it system wide:

​	或者，要在系统范围内安装它：

```sh
$ sudo python -m pip install grpcio
```

#### gRPC tools gRPC 工具

Python’s gRPC tools include the protocol buffer compiler `protoc` and the special plugin for generating server and client code from `.proto` service definitions. For the first part of our quick-start example, we’ve already generated the server and client stubs from [helloworld.proto](https://github.com/grpc/grpc/tree/v1.60.0/examples/protos/helloworld.proto), but you’ll need the tools for the rest of our quick start, as well as later tutorials and your own projects.

​	Python 的 gRPC 工具包括协议缓冲区编译器 `protoc` 和用于从 `.proto` 服务定义生成服务器和客户端代码的特殊插件。对于快速入门示例的第一部分，我们已经从 helloworld.proto 生成了服务器和客户端存根，但您还需要工具来完成我们的快速入门，以及后面的教程和您自己的项目。

To install gRPC tools, run:

​	要安装 gRPC 工具，请运行：

```sh
$ python -m pip install grpcio-tools
```

### Download the example 下载示例

You’ll need a local copy of the example code to work through this quick start. Download the example code from our GitHub repository (the following command clones the entire repository, but you just need the examples for this quick start and other tutorials):

​	您需要示例代码的本地副本才能完成此快速入门。从我们的 GitHub 存储库下载示例代码（以下命令克隆整个存储库，但您只需要此快速入门和其他教程的示例）：

```sh
# Clone the repository to get the example code:
$ git clone -b v1.60.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc
# Navigate to the "hello, world" Python example:
$ cd grpc/examples/python/helloworld
```

### Run a gRPC application 运行 gRPC 应用程序

From the `examples/python/helloworld` directory:

​	从 `examples/python/helloworld` 目录：

1. Run the server: 
   ​	运行服务器：

   ```sh
   $ python greeter_server.py
   ```

2. From another terminal, run the client:

   ​	从另一个终端运行客户端：

   ```sh
   $ python greeter_client.py
   ```

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

### Update the gRPC service 更新 gRPC 服务

Now let’s look at how to update the application with an extra method on the server for the client to call. Our gRPC service is defined using protocol buffers; you can find out lots more about how to define a service in a `.proto` file in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Basics tutorial]({{< ref "/Languages/Python/Basicstutorial" >}}). For now all you need to know is that both the server and the client “stub” have a `SayHello` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that this method is defined like this:

​	现在让我们看看如何使用服务器上的额外方法更新应用程序以供客户端调用。我们的 gRPC 服务使用协议缓冲区定义；您可以在 gRPC 简介和基础教程中找到有关如何在 `.proto` 文件中定义服务的更多信息。现在您只需要知道服务器和客户端“存根”都具有 `SayHello` RPC 方法，该方法从客户端获取 `HelloRequest` 参数并从服务器返回 `HelloReply` ，并且该方法的定义如下：

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

### Generate gRPC code 生成 gRPC 代码

Next we need to update the gRPC code used by our application to use the new service definition.

​	接下来，我们需要更新应用程序使用的 gRPC 代码以使用新的服务定义。

From the `examples/python/helloworld` directory, run:

​	从 `examples/python/helloworld` 目录运行：

```sh
$ python -m grpc_tools.protoc -I../../protos --python_out=. --pyi_out=. --grpc_python_out=. ../../protos/helloworld.proto
```

This regenerates `helloworld_pb2.py` which contains our generated request and response classes and `helloworld_pb2_grpc.py` which contains our generated client and server classes.

​	这会重新生成 `helloworld_pb2.py` ，其中包含我们生成的请求和响应类，以及 `helloworld_pb2_grpc.py` ，其中包含我们生成的客户端和服务器类。

### Update and run the application 更新并运行应用程序

We now have new generated server and client code, but we still need to implement and call the new method in the human-written parts of our example application.

​	我们现在有了新的生成的服务器和客户端代码，但我们仍然需要在示例应用程序的手写部分中实现和调用新方法。

#### Update the server 从示例的根目录

In the same directory, open `greeter_server.py`. Implement the new method like this:

​	在同一目录中，打开 `greeter_server.py` 。像这样实现新方法：

```py
class Greeter(helloworld_pb2_grpc.GreeterServicer):

    def SayHello(self, request, context):
        return helloworld_pb2.HelloReply(message=f"Hello, {request.name}!")

    def SayHelloAgain(self, request, context):
        return helloworld_pb2.HelloReply(message=f"Hello again, {request.name}!")
...
```

#### Update the client 更新客户端

In the same directory, open `greeter_client.py`. Call the new method like this:

​	在同一目录中，打开 `greeter_client.py` 。像这样调用新方法：

```py
def run():
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = helloworld_pb2_grpc.GreeterStub(channel)
        response = stub.SayHello(helloworld_pb2.HelloRequest(name='you'))
        print("Greeter client received: " + response.message)
        response = stub.SayHelloAgain(helloworld_pb2.HelloRequest(name='you'))
        print("Greeter client received: " + response.message)
```

#### Run! 运行！

Just like we did before, from the `examples/python/helloworld` directory:

​	就像我们之前从 `examples/python/helloworld` 目录中所做的那样：

1. Run the server: 
   ​	运行服务器：

   ```sh
   $ python greeter_server.py
   ```

2. From another terminal, run the client:

   ​	从另一个终端运行客户端：

   ```sh
   $ python greeter_client.py
   ```

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Languages/Python/Basicstutorial" >}}).
  完成基础知识教程。
- Explore the [API reference]({{< ref "/Languages/Python/API" >}}).
  探索 API 参考。
