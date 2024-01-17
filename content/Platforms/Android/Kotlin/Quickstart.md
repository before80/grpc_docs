+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/platforms/android/kotlin/quickstart/](https://grpc.io/docs/platforms/android/kotlin/quickstart/)

# Quick start 快速入门

This guide gets you started with Kotlin gRPC on Android with a simple working example.

​	本指南通过一个简单的示例帮助您开始在 Android 上使用 Kotlin gRPC。



### Prerequisites 先决条件

- [Kotlin](https://kotlinlang.org/) version 1.3 or higher

  ​	Kotlin 版本 1.3 或更高

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

gRPC Kotlin does not support running a server on an Android device. For this quick start, the Android client app will connect to a server running on your local (non-Android) computer.
gRPC Kotlin 不支持在 Android 设备上运行服务器。对于此快速入门，Android 客户端应用将连接到在本地（非 Android）计算机上运行的服务器。

### Get the example code 获取示例代码

The example code is part of the [grpc-kotlin](https://github.com/grpc/grpc-kotlin) repo.

​	示例代码是 grpc-kotlin 代码库的一部分。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-kotlin/archive/master.zip) and unzip it, or clone the repo:

   ​	以 zip 文件形式下载代码库并解压缩，或克隆代码库：

   ```sh
   $ git clone https://github.com/grpc/grpc-kotlin
   ```

2. Change to the examples directory:

   ​	更改到 examples 目录：

   ```sh
   $ cd grpc-kotlin/examples
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
   $ ./server/build/install/server/bin/hello-world-server
   Server started, listening on 50051
   ```

3. From another terminal, build the client and install it on your device:

   ​	从另一个终端构建客户端并将其安装到您的设备上：

   ```sh
   $ ./gradlew :android:installDebug
   ```

4. Launch the client app from your device.

   ​	从您的设备启动客户端应用。

5. Type “Alice” in the **Name** box and click **Send**. You’ll see the following response:

   ​	在“姓名”框中输入“Alice”，然后点击“发送”。您将看到以下响应：

   ```nocode
   Hello Alice
   ```

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

### Update the gRPC service 更新 gRPC 服务

In this section you’ll update the application by adding an extra server method. The gRPC service is defined using [protocol buffers](https://developers.google.com/protocol-buffers). To learn more about how to define a service in a `.proto` file see [Basics tutorial]({{< ref "/Languages/Kotlin/Basicstutorial" >}}). For now, all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

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

Follow the instructions given in [Update the server]({{< ref "/Languages/Kotlin/Quickstart#update-the-server" >}}) of the Kotlin quick start page.

​	按照 Kotlin 快速入门页面的“更新服务器”中给出的说明进行操作。

#### Update the client 更新客户端

Follow these steps: 
​	请按照以下步骤操作：

1. Open `helloworld/MainActivity.kt` from the [client/src/main/kotlin/io/grpc/examples](https://github.com/grpc/grpc-kotlin/blob/master/examples/android/src/main/kotlin/io/grpc/examples) folder.

   ​	从 client/src/main/kotlin/io/grpc/examples 文件夹中打开 `helloworld/MainActivity.kt` 。

2. Locate the function containing the call to `sayHello()`. You’ll see these lines of code:

   ​	找到包含对 `sayHello()` 的调用的函数。您将看到以下代码行：

   ```kotlin
   val response = greeter.sayHello(request)
   responseText.text = response.message
   ```

3. Add a call to `sayHelloAgain()` and change how the response message is created. Replace the lines of code above with the following:

   ​	添加对 `sayHelloAgain()` 的调用，并更改响应消息的创建方式。用以下内容替换上述代码行：

   ```kotlin
   val response = greeter.sayHello(request)
   val againResponse = greeter.sayHelloAgain(request)
   val message = "${response.message}\n${againResponse.message}"
   responseText.text = message
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
   $ ./server/build/install/server/bin/hello-world-server
   Server started, listening on 50051
   ```

3. From another terminal, build the client and install it on your device:

   ​	从另一个终端构建客户端并将其安装到您的设备上：

   ```sh
   $ ./gradlew :android:installDebug
   ```

4. Launch the client app from your device.

   ​	从您的设备启动客户端应用。

5. Type “Alice” in the **Message** box and click **Send**. You’ll see the following response:

   ​	在消息框中输入“Alice”，然后点击发送。您将看到以下回复：

   ```nocode
   Hello Alice
   Hello again Alice
   ```

### What’s next 下一步

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/WhatisgRPC/Coreconcepts" >}}).
  在 gRPC 简介和核心概念中了解 gRPC 的工作原理。
- Work through the [Basics tutorial]({{< ref "/Languages/Kotlin/Basicstutorial" >}}) for Kotlin/JVM.
  完成 Kotlin/JVM 的基础知识教程。
- Explore the [API reference]({{< ref "/Platforms/Android/Kotlin/API" >}}).
  探索 API 参考。
