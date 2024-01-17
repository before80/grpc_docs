+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/objective-c/quickstart/](https://grpc.io/docs/languages/objective-c/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC on the iOS platform in Objective-C with a simple working example.

​	本指南通过一个简单的示例，帮助您在 iOS 平台上使用 Objective-C 开始使用 gRPC。



### Before you begin 开始之前

#### System requirements 系统要求

- macOS version 10.11 (El Capitan) or higher
  macOS 版本 10.11（El Capitan）或更高版本
- iOS version 7.0 or higher
  iOS 版本 7.0 或更高版本

#### Prerequisites 先决条件

- CocoaPods version 1.0 or higher

  ​	CocoaPods 版本 1.0 或更高版本

  Check the status and version of CocoaPods on your system:

  ​	检查您系统上 CocoaPods 的状态和版本：

  ```sh
  $ pod --version
  ```

  If CocoaPods is not installed, follow the [CocoaPods install instructions](https://cocoapods.org/).

  ​	如果未安装 CocoaPods，请按照 CocoaPods 安装说明进行操作。

- Xcode version 7.2 or higher

  ​	Xcode 版本 7.2 或更高版本

  Check your Xcode version by running Xcode from Lauchpad, then select **Xcode > About Xcode** in the menu.

  ​	通过从启动板运行 Xcode，然后在菜单中选择 Xcode > 关于 Xcode 来检查您的 Xcode 版本。

  Make sure the command line developer tools are installed:

  ​	确保已安装命令行开发者工具：

  ```sh
  $ xcode-select --install
  ```

- [Homebrew](https://brew.sh/)

- `autoconf`, `automake`, `libtool`, `pkg-config`

  ```sh
  $ brew install autoconf automake libtool pkg-config
  ```

### Download the example 下载示例

You’ll need a local copy of the sample app source code to work through this Quickstart. Copy the source code from GitHub [repository](https://github.com/grpc/grpc):

​	要完成此快速入门，您需要一个示例应用源代码的本地副本。从 GitHub 存储库复制源代码：

```sh
$ git clone --recursive -b v1.60.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc
```

### Install gRPC plugins and libraries 安装 gRPC 插件和库

```sh
$ cd grpc
$ make
$ [sudo] make install
```

### Install protoc compiler 安装 protoc 编译器

```sh
$ brew tap grpc/grpc
$ brew install protobuf
```

### Run the server: 运行服务器：

For this sample app, we need a gRPC server running on the local machine. gRPC Objective-C API supports creating gRPC clients but not gRPC servers. Therefore instead we build and run the C++ server in the same repository:

​	对于此示例应用，我们需要在本地计算机上运行 gRPC 服务器。gRPC Objective-C API 支持创建 gRPC 客户端，但不支持 gRPC 服务器。因此，我们转而构建并运行同一存储库中的 C++ 服务器：

```sh
$ cd examples/cpp/helloworld
$ make
$ ./greeter_server &
```

### Run the client: 运行客户端：

#### Generate client libraries and dependencies 生成客户端库和依赖项

Have CocoaPods generate and install the client library from our .proto files, as well as installing several dependencies:

​	让 CocoaPods 从我们的 .proto 文件生成并安装客户端库，以及安装多个依赖项：

```sh
$ cd ../../objective-c/helloworld
$ pod install
```

(This might have to compile OpenSSL, which takes around 15 minutes if Cocoapods doesn’t have it yet on your computer’s cache.)

​	（这可能需要编译 OpenSSL，如果 Cocoapods 尚未在您计算机的缓存中，则大约需要 15 分钟。）

#### Run the client app 运行客户端应用

Open the Xcode workspace created by CocoaPods:

​	打开 CocoaPods 创建的 Xcode 工作区：

```sh
$ open HelloWorld.xcworkspace
```

This will open the app project with Xcode. Run the app in an iOS simulator by pressing the Run button on the top left corner of Xcode window. You can check the calling code in `main.m` and see the results in Xcode’s console.

​	这将使用 Xcode 打开应用项目。通过按 Xcode 窗口左上角的运行按钮，在 iOS 模拟器中运行应用。您可以在 `main.m` 中检查调用代码，并在 Xcode 的控制台中查看结果。

The code sends a `HLWHelloRequest` containing the string “Objective-C” to a local server. The server responds with a `HLWHelloResponse`, which contains a string “Hello Objective-C” that is then output to the console.

​	该代码将包含字符串“Objective-C”的 `HLWHelloRequest` 发送到本地服务器。服务器使用 `HLWHelloResponse` 响应，其中包含字符串“Hello Objective-C”，然后将其输出到控制台。

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

### Update the gRPC service 更新 gRPC 服务

Now let’s look at how to update the application with an extra method on the server for the client to call. Our gRPC service is defined using Protocol Buffers; you can find out lots more about how to define a service in a `.proto` file in Protocol Buffers [website](https://protobuf.dev/). For now all you need to know is that both the server and the client “stub” have a `SayHello` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloResponse` from the server, and that this method is defined like this:

​	现在，我们来看看如何使用服务器上的额外方法更新应用程序，以便客户端调用。我们的 gRPC 服务使用 Protocol Buffers 定义；您可以在 Protocol Buffers 网站上找到有关如何在 `.proto` 文件中定义服务的更多信息。现在，您只需要知道服务器和客户端“存根”都具有 `SayHello` RPC 方法，该方法从客户端获取 `HelloRequest` 参数并从服务器返回 `HelloResponse` ，并且该方法的定义如下：

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

### Update the client and server 更新客户端和服务器

We now have a new gRPC service definition, but we still need to implement and call the new method in the human-written parts of our example application.

​	我们现在有了新的 gRPC 服务定义，但我们仍然需要在示例应用程序的人工编写部分中实现和调用新方法。

#### Update the server 从示例的根目录

As you remember, gRPC doesn’t provide a server API for Objective-C. Instead, we need to update the C++ sample server. Open `examples/cpp/helloworld/greeter_server.cc`. Implement the new method like this:

​	如你所知，gRPC 并未提供适用于 Objective-C 的服务器 API。相反，我们需要更新 C++ 示例服务器。打开 `examples/cpp/helloworld/greeter_server.cc` 。像这样实现新方法：

```c++
class GreeterServiceImpl final : public Greeter::Service {
  Status SayHello(ServerContext* context, const HelloRequest* request,
                  HelloReply* reply) override {
    std::string prefix("Hello ");
    reply->set_message(prefix + request->name());
    return Status::OK;
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

Edit the main function in `examples/objective-c/helloworld/main.m` to call the new method like this:

​	在 `examples/objective-c/helloworld/main.m` 中编辑 main 函数，像这样调用新方法：

```c
int main(int argc, char * argv[]) {
  @autoreleasepool {
    HLWGreeter *client = [[HLWGreeter alloc] initWithHost:kHostAddress];

    HLWHelloRequest *request = [HLWHelloRequest message];
    request.name = @"Objective-C";

    GRPCMutableCallOptions *options = [[GRPCMutableCallOptions alloc] init];
    // this example does not use TLS (secure channel); use insecure channel instead
    options.transport = GRPCDefaultTransportImplList.core_insecure;
    options.userAgentPrefix = @"HelloWorld/1.0";

    [[client sayHelloWithMessage:request
                 responseHandler:[[HLWResponseHandler alloc] init]
                     callOptions:options] start];
    [[client sayHelloAgainWithMessage:request
                      responseHandler:[[HLWResponseHandler alloc] init]
                          callOptions:options] start];

    return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
  }
}
```

### Build and run 构建并运行

First terminate the server process already running in the background:

​	首先终止已经在后台运行的服务器进程：

```sh
$ pkill greeter_server
```

Then in directory `examples/cpp/helloworld`, build and run the updated server with the following commands:

​	然后在目录 `examples/cpp/helloworld` 中，使用以下命令构建并运行更新的服务器：

```sh
$ make
$ ./greeter_server &
```

Change directory to `examples/objective-c/helloworld`, then clean up and reinstall Pods for the client app with the following commands:

​	将目录更改为 `examples/objective-c/helloworld` ，然后使用以下命令清理并重新安装客户端应用的 Pods：

```sh
$ rm -Rf Pods
$ rm Podfile.lock
$ rm -Rf HelloWorld.xcworkspace
$ pod install
```

This regenerates files in `Pods/HelloWorld` based on the new proto file we wrote above. Open the client Xcode project in Xcode:

​	这会根据我们上面编写的新的 proto 文件重新生成 `Pods/HelloWorld` 中的文件。在 Xcode 中打开客户端 Xcode 项目：

```sh
$ open HelloWorld.xcworkspace
```

and run the client app. If you look at the console messages, You’ll see two RPC calls, one to SayHello and one to SayHelloAgain.

​	并运行客户端应用。如果你查看控制台消息，你将看到两个 RPC 调用，一个用于 SayHello，另一个用于 SayHelloAgain。

### Troubleshooting 故障排除

- When installing CocoaPods, error `activesupport requires Ruby version >= 2.2.2` 安装 CocoaPods 时，错误 `activesupport requires Ruby version >= 2.2.2`

  Install an older version of `activesupport`, then install CocoaPods:  ​	安装旧版本的 `activesupport` ，然后安装 CocoaPods：`$ [sudo] gem install activesupport -v 4.2.6 $ [sudo] gem install cocoapods `

- When installing dependencies with CocoaPods, error `Unable to find a specification for !ProtoCompiler-gRPCPlugin` 使用 CocoaPods 安装依赖项时，错误 `Unable to find a specification for !ProtoCompiler-gRPCPlugin`

  Update the local clone of spec repo by running `pod repo update`  ​	通过运行 `pod repo update` 更新本地克隆的规范仓库

- Compiler error when compiling `objective_c_plugin.cc` 编译 `objective_c_plugin.cc` 时出现编译器错误

  Removing `protobuf` package with Homebrew before building gRPC may solve this problem. We are working on a more elegant fix.  ​	在构建 gRPC 之前使用 Homebrew 删除 `protobuf` 软件包可能会解决此问题。我们正在努力寻找更优雅的修复方法。

- When building HellowWorld, error `ld: unknown option: --no-as-needed` 构建 HellowWorld 时，错误 `ld: unknown option: --no-as-needed`

  This problem is due to linker `ld` in Apple LLVM not supporting the `--no-as-needed` option. We are working on a fix right now and will merge the fix very soon.  ​	此问题是由于 Apple LLVM 中的链接器 `ld` 不支持 `--no-as-needed` 选项。我们目前正在努力修复此问题，并将很快合并修复。

- When building grpc, error `cannot find install-sh install.sh or shtool` 构建 grpc 时，错误 `cannot find install-sh install.sh or shtool`

  Remove the gRPC directory, clone a new one and try again. It is likely that some auto generated files are corrupt; remove and rebuild may solve the problem.  ​	删除 gRPC 目录，克隆一个新的目录，然后重试。某些自动生成的文件可能已损坏；删除并重新构建可能会解决此问题。

- When building grpc, error `Can't exec "aclocal"` 构建 grpc 时，错误 `Can't exec "aclocal"`

  The package `automake` is missing. Install `automake` should solve this problem.  ​	缺少软件包 `automake` 。安装 `automake` 应该可以解决此问题。

- When building grpc, error `possibly undefined macro: AC_PROG_LIBTOOL` 在构建 grpc 时，错误 `possibly undefined macro: AC_PROG_LIBTOOL`

  The package `libtool` is missing. Install `libtool` should solve this problem.  ​	缺少软件包 `libtool` 。安装 `libtool` 应该可以解决此问题。

- When building grpc, error `cannot find install-sh, install.sh, or shtool` 在构建 grpc 时，错误 `cannot find install-sh, install.sh, or shtool`

  Some of the auto generated files are corrupt. Remove the entire gRPC directory, clone from GitHub, and build again.  ​	某些自动生成的文件已损坏。删除整个 gRPC 目录，从 GitHub 克隆，然后重新构建。

- Cannot find `protoc` when building HelloWorld 在构建 HelloWorld 时找不到 `protoc`

  Run `brew install protobuf` to get the `protoc` compiler.  ​	运行 `brew install protobuf` 以获取 `protoc` 编译器。

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Languages/Objective-C/Basicstutorial" >}}).
  完成基础知识教程。
- Explore the [API reference]({{< ref "/Languages/Objective-C/API" >}}).
  探索 API 参考。
