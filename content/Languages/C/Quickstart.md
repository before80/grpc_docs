+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/cpp/quickstart/](https://grpc.io/docs/languages/cpp/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC in C++ with a simple working example.

​	本指南通过一个简单的示例帮助您开始使用 C++ 中的 gRPC。



In the C++ world, there’s no universally accepted standard for managing project dependencies. You need to build and install gRPC before building and running this quick start’s Hello World example.

​	在 C++ 世界中，没有普遍接受的管理项目依赖项的标准。您需要在构建并运行此快速入门 Hello World 示例之前构建并安装 gRPC。

### Build and locally install gRPC and Protocol Buffers 构建并本地安装 gRPC 和 Protocol Buffers

The steps in the section explain how to build and locally install gRPC and Protocol Buffers using `cmake`. If you’d rather use [bazel](https://www.bazel.build/), see [Building from source](https://github.com/grpc/grpc/blob/master/BUILDING.md#build-from-source).

​	本部分中的步骤说明了如何使用 `cmake` 构建并本地安装 gRPC 和 Protocol Buffers。如果您更愿意使用 bazel，请参阅从源代码构建。

#### Setup 设置

Choose a directory to hold locally installed packages. This page assumes that the environment variable `MY_INSTALL_DIR` holds this directory path. For example:

​	选择一个目录来保存本地安装的软件包。本页假设环境变量 `MY_INSTALL_DIR` 保存此目录路径。例如：

```sh
$ export MY_INSTALL_DIR=$HOME/.local
```

Ensure that the directory exists:

​	确保目录存在：

```sh
$ mkdir -p $MY_INSTALL_DIR
```

Add the local `bin` folder to your path variable, for example:

​	将本地 `bin` 文件夹添加到您的路径变量，例如：

```sh
$ export PATH="$MY_INSTALL_DIR/bin:$PATH"
```

#### Install cmake 安装 cmake

You need version 3.13 or later of `cmake`. Install it by following these instructions:

​	您需要 `cmake` 的 3.13 或更高版本。按照以下说明进行安装：

- Linux

  ```sh
  $ sudo apt install -y cmake
  ```

- macOS: 
  ​	macOS：

  ```sh
  $ brew install cmake
  ```

- For general `cmake` installation instructions, see [Installing CMake](https://cmake.org/install).

  ​	有关常规 `cmake` 安装说明，请参阅安装 CMake。

Check the version of `cmake`:

​	检查 `cmake` 的版本：

```sh
$ cmake --version
cmake version 3.19.6
```

Under Linux, the version of the system-wide `cmake` can often be too old. You can install a more recent version into your local installation directory as follows:

​	在 Linux 中，系统范围的 `cmake` 的版本通常可能太旧。您可以按照以下步骤将较新版本安装到您的本地安装目录中：

```sh
$ wget -q -O cmake-linux.sh https://github.com/Kitware/CMake/releases/download/v3.19.6/cmake-3.19.6-Linux-x86_64.sh
$ sh cmake-linux.sh -- --skip-license --prefix=$MY_INSTALL_DIR
$ rm cmake-linux.sh
```

#### Install other required tools 安装其他必需的工具

Install the basic tools required to build gRPC:

​	安装构建 gRPC 所需的基本工具：

- Linux

  ```sh
  $ sudo apt install -y build-essential autoconf libtool pkg-config
  ```

- macOS: 
  ​	macOS：

  ```sh
  $ brew install autoconf automake libtool pkg-config
  ```

#### Clone the `grpc` repo 克隆 `grpc` 代码库

Clone the `grpc` repo and its submodules:

​	克隆 `grpc` 代码库及其子模块：

```sh
$ git clone --recurse-submodules -b v1.60.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc
```

#### Build and install gRPC and Protocol Buffers 构建并安装 gRPC 和 Protocol Buffers

While not mandatory, gRPC applications usually leverage [Protocol Buffers](https://developers.google.com/protocol-buffers) for service definitions and data serialization, and the example code uses [proto3](https://protobuf.dev/programming-guides/proto3).

​	虽然不是强制性的，但 gRPC 应用程序通常利用 Protocol Buffers 进行服务定义和数据序列化，示例代码使用 proto3。

The following commands build and locally install gRPC and Protocol Buffers:

​	以下命令构建并本地安装 gRPC 和 Protocol Buffers：

```sh
$ cd grpc
$ mkdir -p cmake/build
$ pushd cmake/build
$ cmake -DgRPC_INSTALL=ON \
      -DgRPC_BUILD_TESTS=OFF \
      -DCMAKE_INSTALL_PREFIX=$MY_INSTALL_DIR \
      ../..
$ make -j 4
$ make install
$ popd
```

#### Important 重要

We **strongly** encourage you to install gRPC *locally* — using an appropriately set `CMAKE_INSTALL_PREFIX` — because there is no easy way to uninstall gRPC after you’ve installed it globally.
我们强烈建议您本地安装 gRPC — 使用适当设置的 `CMAKE_INSTALL_PREFIX` — 因为在全局安装后，没有简单的方法来卸载 gRPC。

More information: 
​	更多信息：

- You can find a complete set of instructions for building gRPC C++ in [Building from source](https://github.com/grpc/grpc/blob/master/BUILDING.md#build-from-source).
  您可以在从源代码构建中找到构建 gRPC C++ 的完整说明。
- For general instructions on how to add gRPC as a dependency to your C++ project, see [Start using gRPC C++](https://github.com/grpc/grpc/tree/master/src/cpp#to-start-using-grpc-c).
  有关如何将 gRPC 添加为 C++ 项目的依赖项的常规说明，请参阅开始使用 gRPC C++。

### Build the example 构建示例

The example code is part of the `grpc` repo source, which you cloned as part of the steps of the previous section.

​	示例代码是 `grpc` 回购源的一部分，您已在上一部分的步骤中克隆了该源。

1. Change to the example’s directory:

   ​	更改为示例目录：

   ```sh
   $ cd examples/cpp/helloworld
   ```

2. Build the example using `cmake`:

   ​	使用 `cmake` 构建示例：

   ```sh
   $ mkdir -p cmake/build
   $ pushd cmake/build
   $ cmake -DCMAKE_PREFIX_PATH=$MY_INSTALL_DIR ../..
   $ make -j 4
   ```

   #### Note 注意

   **Getting build failures? 遇到构建失败？** Most issues, at this point, are the result of a faulty installation. Make sure you have the right version of `cmake`, and carefully recheck your installation.
   此时，大多数问题都是由于安装错误造成的。确保您拥有 `cmake` 的正确版本，并仔细重新检查您的安装。

### Try it! 试试看！

Run the example from the example **build** directory `examples/cpp/helloworld/cmake/build`:

​	从示例构建目录 `examples/cpp/helloworld/cmake/build` 运行示例：

1. Run the server: 
   ​	运行服务器：

   ```sh
   $ ./greeter_server
   ```

2. From a different terminal, run the client and see the client output:

   ​	从另一个终端运行客户端并查看客户端输出：

   ```sh
   $ ./greeter_client
   Greeter received: Hello world
   ```

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

### Update the gRPC service 更新 gRPC 服务

Now let’s look at how to update the application with an extra method on the server for the client to call. Our gRPC service is defined using protocol buffers; you can find out lots more about how to define a service in a `.proto` file in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Basics tutorial]({{< ref "/Languages/C/Basicstutorial" >}}). For now all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that this method is defined like this:

​	现在让我们看看如何使用服务器上的额外方法更新应用程序以供客户端调用。我们的 gRPC 服务使用协议缓冲区定义；您可以在 gRPC 简介和基础教程中找到有关如何在 `.proto` 文件中定义服务的更多信息。现在您只需要知道服务器和客户端存根都具有 `SayHello()` RPC 方法，该方法从客户端获取 `HelloRequest` 参数并从服务器返回 `HelloReply` ，并且该方法的定义如下：

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

Open [examples/protos/helloworld.proto](https://github.com/grpc/grpc/blob/v1.60.0/examples/protos/helloworld.proto) and add a new `SayHelloAgain()` method, with the same request and response types:

​	打开 examples/protos/helloworld.proto 并添加一个新的 `SayHelloAgain()` 方法，具有相同的请求和响应类型：

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

Before you can use the new service method, you need to recompile the updated proto file.

​	在您使用新的服务方法之前，您需要重新编译更新的 proto 文件。

From the example **build** directory `examples/cpp/helloworld/cmake/build`, run:

​	从示例构建目录 `examples/cpp/helloworld/cmake/build` 运行：

```sh
$ make -j 4
```

This regenerates `helloworld.pb.{h,cc}` and `helloworld.grpc.pb.{h,cc}`, which contains the generated client and server classes, as well as classes for populating, serializing, and retrieving our request and response types.

​	这会重新生成 `helloworld.pb.{h,cc}` 和 `helloworld.grpc.pb.{h,cc}` ，其中包含生成的客户端和服务器类，以及用于填充、序列化和检索我们的请求和响应类型的类。

### Update and run the application 更新并运行应用程序

You have new generated server and client code, but you still need to implement and call the new method in the human-written parts of our example application.

​	您有新的生成的服务器和客户端代码，但您仍然需要在示例应用程序的手写部分中实现和调用新方法。

#### Update the server 从示例的根目录

Open `greeter_server.cc` from the example’s root directory. Implement the new method like this:

​	Open `greeter_server.cc` 更新服务器。像这样实现新方法：

```c++
class GreeterServiceImpl final : public Greeter::Service {
  Status SayHello(ServerContext* context, const HelloRequest* request,
                  HelloReply* reply) override {
     // ...
  }

  Status SayHelloAgain(ServerContext* context, const HelloRequest* request,
                       HelloReply* reply) override {
    std::string prefix("Hello again ");
    reply->set_message(prefix + request->name());
    return Status::OK;
  }
};
```

#### Update the client 更新客户端

A new `SayHelloAgain()` method is now available in the stub. We’ll follow the same pattern as for the already present `SayHello()` and add a new `SayHelloAgain()` method to `GreeterClient`:

​	存根中现在提供了一个新 `SayHelloAgain()` 方法。我们将遵循与已存在的 `SayHello()` 相同的模式，并向 `GreeterClient` 添加一个新的 `SayHelloAgain()` 方法：

```c++
class GreeterClient {
 public:
  // ...
  std::string SayHello(const std::string& user) {
     // ...
  }

  std::string SayHelloAgain(const std::string& user) {
    // Follows the same pattern as SayHello.
    HelloRequest request;
    request.set_name(user);
    HelloReply reply;
    ClientContext context;

    // Here we can use the stub's newly available method we just added.
    Status status = stub_->SayHelloAgain(&context, request, &reply);
    if (status.ok()) {
      return reply.message();
    } else {
      std::cout << status.error_code() << ": " << status.error_message()
                << std::endl;
      return "RPC failed";
    }
  }
```

Finally, invoke this new method in `main()`:

​	最后，在 `main()` 中调用此新方法：

```c++
int main(int argc, char** argv) {
  // ...
  std::string reply = greeter.SayHello(user);
  std::cout << "Greeter received: " << reply << std::endl;

  reply = greeter.SayHelloAgain(user);
  std::cout << "Greeter received: " << reply << std::endl;

  return 0;
}
```

#### Run! 运行！

Run the client and server like you did before. Execute the following commands from the example **build** directory `examples/cpp/helloworld/cmake/build`:

​	像以前一样运行客户端和服务器。从示例构建目录 `examples/cpp/helloworld/cmake/build` 执行以下命令：

1. Build the client and server after having made changes:

   ​	在做出更改后构建客户端和服务器：

   ```sh
   $ make -j 4
   ```

2. Run the server: 
   ​	运行服务器：

   ```sh
   $ ./greeter_server
   ```

3. On a different terminal, run the client:

   ​	在另一个终端上，运行客户端：

   ```sh
   $ ./greeter_client
   ```

   You’ll see the following output:

   ​	您将看到以下输出：

   ```nocode
   Greeter received: Hello world
   Greeter received: Hello again world
   ```

#### Note 注意

Interested in an **asynchronous** version of the client and server? You’ll find the `greeter_async_{client,server}.cc` files in the [example’s source directory](https://github.com/grpc/grpc/tree/master/examples/cpp/helloworld).
对客户端和服务器的异步版本感兴趣？您可以在示例的源目录中找到 `greeter_async_{client,server}.cc` 文件。

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Languages/C/Basicstutorial" >}}).
  完成基础知识教程。
- Explore the [API reference]({{< ref "/Languages/C/API" >}}).
  探索 API 参考。
