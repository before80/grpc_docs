+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/platforms/android/java/quickstart/](https://grpc.io/docs/platforms/android/java/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC in Android Java with a simple working example.

​	本指南通过一个简单的示例帮助您开始在 Android Java 中使用 gRPC。



### Prerequisites 先决条件

- [JDK](https://jdk.java.net/) version 7 or higher

  ​	JDK 版本 7 或更高

- Android SDK, API level 16 or higher

  ​	Android SDK，API 级别 16 或更高

  1. Install [Android Studio](https://developer.android.com/studio/index.html#downloads) or the Android [command-line tools](https://developer.android.com/studio/index.html#command-tools).

     ​	安装 Android Studio 或 Android 命令行工具。

  2. Let other tools and scripts know where to find your Android SDK by setting the following environment variable:

     ​	通过设置以下环境变量，让其他工具和脚本知道在哪里可以找到您的 Android SDK：

     ```sh
     $ export ANDROID_SDK_ROOT="<path-to-your-android-sdk>"
     ```

- An android device set up for [USB debugging](https://developer.android.com/studio/command-line/adb.html#Enabling) or an [Android Virtual Device](https://developer.android.com/studio/run/managing-avds.html)

  ​	为 USB 调试设置的 Android 设备或 Android 虚拟设备

#### Note 注意

gRPC Java does not support running a server on an Android device. For this quick start, the Android client app will connect to a server running on your local (non-Android) computer.
gRPC Java 不支持在 Android 设备上运行服务器。对于此快速入门，Android 客户端应用将连接到在您的本地（非 Android）计算机上运行的服务器。

### Get the example code 获取示例代码

The example code is part of the [grpc-java](https://github.com/grpc/grpc-java) repo.

​	示例代码是 grpc-java 代码库的一部分。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-java/archive/v1.61.0.zip) and unzip it, or clone the repo:

   ​	以 zip 文件形式下载代码库并解压缩，或克隆代码库：

   ```sh
   $ git clone -b v1.61.0 https://github.com/grpc/grpc-java
   ```

2. Change to the examples directory:

   ​	更改到 examples 目录：

   ```sh
   $ cd grpc-java/examples
   ```

### Run the example 运行示例

1. Compile the server: 
   ​	编译服务器：

   ```sh
   $ ./gradlew installDist
   ```

2. Run the server: 
   ​	运行服务器：

   ```sh
   $ ./build/install/examples/bin/hello-world-server
   INFO: Server started, listening on 50051
   ```

3. From another terminal, build the client and install it on your device:

   ​	从另一个终端，构建客户端并将其安装到您的设备：

   ```sh
   $ (cd android/helloworld; ../../gradlew installDebug)
   ```

4. Launch the client app from your device.

   ​	从您的设备启动客户端应用。

5. In the client app, enter the server’s **Host** and **Port** information. The values you enter depend on the device kind (real or virtual) — for details, see [Connecting to the server]({{< ref "/Platforms/Android/Java/Quickstart#connecting-to-the-server" >}}) below.

   ​	在客户端应用中，输入服务器的主机和端口信息。您输入的值取决于设备类型（真实或虚拟）——有关详细信息，请参阅下文的连接到服务器。

6. Type “Alice” in the **Message** box and click **Send**. You’ll see the following response:

   ​	在消息框中键入“Alice”，然后点击发送。您将看到以下响应：

   ```nocode
   Hello Alice
   ```

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

#### Note 注意

We’ve omitted timestamps from the client and server trace output shown in this page.
我们已从本页中显示的客户端和服务器跟踪输出中省略了时间戳。

### Update the gRPC service 更新 gRPC 服务

In this section you’ll update the application by adding an extra server method. The gRPC service is defined using [protocol buffers](https://developers.google.com/protocol-buffers). To learn more about how to define a service in a `.proto` file see [Basics tutorial]({{< ref "/Platforms/Android/Java/Basicstutorial" >}}). For now, all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

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

Make the following changes:

​	进行以下更改：

1. Open `src/main/proto/helloworld.proto` and add a new `SayHelloAgain()` method with the same request and response types as `SayHello()`:

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

2. Make the same change to `android/helloworld/app/src/main/proto/helloworld.proto`.

   ​	对 `android/helloworld/app/src/main/proto/helloworld.proto` 进行相同的更改。

Remember to save the files!

​	记得保存文件！

### Update the app 更新应用

When you build the example, the build process regenerates `GreeterGrpc.java`, which contains the generated gRPC client and server classes. This also regenerates classes for populating, serializing, and retrieving our request and response types.

​	在您构建示例时，构建过程会重新生成 `GreeterGrpc.java` ，其中包含生成的 gRPC 客户端和服务器类。这还会重新生成用于填充、序列化和检索我们的请求和响应类型的类。

However, you still need to implement and call the new method in the hand-written parts of the example app.

​	但是，您仍然需要在示例应用的手写部分中实现并调用新方法。

#### Update the server 从示例的根目录

Follow the instructions given in [Update the server]({{< ref "/Languages/Java/Quickstart#update-the-server" >}}) of the Java quick start page.

​	按照 Java 快速入门页面中的更新服务器的说明进行操作。

#### Update the client 更新客户端

Follow these steps: 
​	请按照以下步骤操作：

1. Open `HelloworldActivity.java` from the `android/helloworld/app/src/main/java/io/grpc/helloworldexample` folder.

   ​	从 `android/helloworld/app/src/main/java/io/grpc/helloworldexample` 文件夹中打开 `HelloworldActivity.java` 。

2. Locate the method containing the call to `sayHello()`. You’ll see these lines of code:

   ​	找到包含对 `sayHello()` 的调用的方法。您将看到以下代码行：

   ```java
   HelloReply reply = stub.sayHello(request);
   return reply.getMessage();
   ```

3. Add a call to `sayHelloAgain()` in the `return` statement expression like this:

   ​	在 `return` 语句表达式中添加对 `sayHelloAgain()` 的调用，如下所示：

   ```java
   return reply.getMessage() + "\n" + stub.sayHelloAgain(request).getMessage();
   ```

### Run the updated app 运行更新后的应用

Run the client and server like you did before. Execute the following commands from the `examples` directory:

​	像以前一样运行客户端和服务器。从 `examples` 目录执行以下命令：

1. Compile the server: 
   ​	编译服务器：

   ```sh
   $ ./gradlew installDist
   ```

2. Run the server: 
   ​	运行服务器：

   ```sh
   $ ./build/install/examples/bin/hello-world-server
   INFO: Server started, listening on 50051
   ```

3. From another terminal, build the client and install it on your device:

   ​	从另一个终端构建客户端并将其安装到您的设备上：

   ```sh
   $ (cd android/helloworld; ../../gradlew installDebug)
   ```

4. Launch the client app from your device.

   ​	从您的设备启动客户端应用。

5. In the client app, enter the server’s **Host** and **Port** information. The values you enter depend on the device kind (real or virtual) — for details, see [Connecting to the server]({{< ref "/Platforms/Android/Java/Quickstart#connecting-to-the-server" >}}) below.

   ​	在客户端应用中，输入服务器的主机和端口信息。您输入的值取决于设备类型（真实或虚拟）——有关详细信息，请参阅下面的连接到服务器。

6. Type “Alice” in the **Message** box and click **Send**. You’ll see the following response:

   ​	在消息框中输入“Alice”，然后点击发送。您将看到以下回复：

   ```nocode
   Hello Alice
   Hello again Alice
   ```

### Connecting to the server 正在连接到服务器

#### Connecting from a virtual device 正在从虚拟设备连接

Run the Hello World app on your Android Virtual Device and use the following values:

​	在您的 Android 虚拟设备上运行 Hello World 应用，并使用以下值：

- **Host**: `10.0.2.2` 主机： `10.0.2.2`
- **Port**: 50051 端口：50051

#### Connecting from a physical device 正在从物理设备连接

To run the app on a physical device via USB debugging, you must configure USB port forwarding using the `adb` command as follows:

​	要通过 USB 调试在物理设备上运行该应用，您必须使用 `adb` 命令配置 USB 端口转发，如下所示：

```sh
$ adb reverse tcp:8080 tcp:50051
```

This sets up port forwarding from port `8080` on the device to port `50051` on the connected computer, which is the port that the Hello World server is listening on.

​	这会设置从设备上的端口 `8080` 到连接的计算机上的端口 `50051` 的端口转发，这是 Hello World 服务器正在监听的端口。

In the app, use the following values:

​	在应用中，使用以下值：

- **Host**: `localhost` 主机： `localhost`
- **Port**: 8080 端口：8080

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Platforms/Android/Java/Basicstutorial" >}}).
  完成基础知识教程。
- Explore the [API reference]({{< ref "/Platforms/Android/Java/API" >}}).
  探索 API 参考。
