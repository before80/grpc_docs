+++
title = "简介"
date = 2024-11-19T10:19:42+08:00
weight = 1
type = "docs"
description = "gRPC 和协议缓冲区（Protocol Buffers）的简介。"
isCJKLanguage = true
draft = false

+++

> 原文：[https://grpc.io/docs/what-is-grpc/introduction/](https://grpc.io/docs/what-is-grpc/introduction/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Introduction to gRPC - gRPC 简介

An introduction to gRPC and protocol buffers.

This page introduces you to gRPC and protocol buffers. gRPC can use protocol buffers as both its Interface Definition Language (**IDL**) and as its underlying message interchange format. If you’re new to gRPC and/or protocol buffers, read this! If you just want to dive in and see gRPC in action first, [select a language]({{< ref "/docs/Languages" >}}) and try its **Quick start**.

​	本页面将向您介绍 gRPC 和协议缓冲区。gRPC 可以将协议缓冲区用作其接口定义语言（**IDL**）以及其底层消息交换格式。如果您是 gRPC 和/或协议缓冲区的新手，请阅读本页面！如果您只想快速了解 gRPC 的实际应用，[选择一种语言]({{< ref "/docs/Languages" >}})，然后尝试其 **快速入门**。

{{< youtube "njC24ts24Pg">}}

## 概述 Overview

In gRPC, a client application can directly call a method on a server application on a different machine as if it were a local object, making it easier for you to create distributed applications and services. As in many RPC systems, gRPC is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. On the server side, the server implements this interface and runs a gRPC server to handle client calls. On the client side, the client has a stub (referred to as just a client in some languages) that provides the same methods as the server.

​	在 gRPC 中，客户端应用程序可以像调用本地对象一样直接调用位于另一台机器上的服务器应用程序的方法，这让您更容易创建分布式应用程序和服务。与许多 RPC 系统类似，gRPC 的核心思想是定义服务，指定可以远程调用的方法以及这些方法的参数和返回类型。在服务器端，服务器实现此接口并运行 gRPC 服务器以处理客户端调用。在客户端，客户端有一个存根（在某些语言中称为客户端），它提供与服务器相同的方法。



![Concept Diagram](Introduction_img/landing-2.svg)



gRPC clients and servers can run and talk to each other in a variety of environments - from servers inside Google to your own desktop - and can be written in any of gRPC’s supported languages. So, for example, you can easily create a gRPC server in Java with clients in Go, Python, or Ruby. In addition, the latest Google APIs will have gRPC versions of their interfaces, letting you easily build Google functionality into your applications.

​	gRPC 客户端和服务器可以在多种环境中运行并互相通信——从 Google 内部的服务器到您自己的桌面——并且可以用 gRPC 支持的任何语言编写。例如，您可以轻松地用 Java 创建一个 gRPC 服务器，并用 Go、Python 或 Ruby 编写客户端。此外，最新的 Google API 将提供 gRPC 版本的接口，让您可以轻松地将 Google 功能集成到您的应用程序中。

### 使用协议缓冲区 Working with Protocol Buffers

By default, gRPC uses [Protocol Buffers](https://protobuf.dev/overview), Google’s mature open source mechanism for serializing structured data (although it can be used with other data formats such as JSON). Here’s a quick intro to how it works. If you’re already familiar with protocol buffers, feel free to skip ahead to the next section.

​	默认情况下，gRPC 使用 [协议缓冲区](https://protobuf.dev/overview)，这是 Google 成熟的开源结构化数据序列化机制（尽管它也可以与 JSON 等其他数据格式一起使用）。以下是其工作原理的快速介绍。如果您已熟悉协议缓冲区，可以跳到下一节。

The first step when working with protocol buffers is to define the structure for the data you want to serialize in a *proto file*: this is an ordinary text file with a `.proto` extension. Protocol buffer data is structured as *messages*, where each message is a small logical record of information containing a series of name-value pairs called *fields*. Here’s a simple example:

​	使用协议缓冲区的第一步是定义要序列化的数据结构，这在一个 *proto 文件* 中完成：这是一个扩展名为 `.proto` 的普通文本文件。协议缓冲区数据以 *消息* 的形式结构化，每个消息是一个包含一系列名称-值对（称为 *字段*）的小型逻辑记录。以下是一个简单的示例：

```proto
message Person {
  string name = 1;
  int32 id = 2;
  bool has_ponycopter = 3;
}
```

Then, once you’ve specified your data structures, you use the protocol buffer compiler `protoc` to generate data access classes in your preferred language(s) from your proto definition. These provide simple accessors for each field, like `name()` and `set_name()`, as well as methods to serialize/parse the whole structure to/from raw bytes. So, for instance, if your chosen language is C++, running the compiler on the example above will generate a class called `Person`. You can then use this class in your application to populate, serialize, and retrieve `Person` protocol buffer messages.

​	然后，一旦您定义了数据结构，就可以使用协议缓冲区编译器 `protoc` 从您的 proto 定义中生成您首选语言的数据访问类。这些类提供了每个字段的简单访问器，如 `name()` 和 `set_name()`，以及将整个结构序列化/解析为原始字节的功能。例如，如果您的首选语言是 C++，运行上述示例的编译器将生成一个名为 `Person` 的类。然后，您可以在应用程序中使用此类来填充、序列化和检索 `Person` 协议缓冲区消息。

You define gRPC services in ordinary proto files, with RPC method parameters and return types specified as protocol buffer messages:

​	在普通的 proto 文件中定义 gRPC 服务，同时用协议缓冲区消息指定 RPC 方法参数和返回类型：

```proto
// The greeter service definition.
// Greeter 服务定义。
service Greeter {
  // Sends a greeting
  // 发送问候
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
// 请求消息，包含用户的名称。
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
// 响应消息，包含问候内容。
message HelloReply {
  string message = 1;
}
```

gRPC uses `protoc` with a special gRPC plugin to generate code from your proto file: you get generated gRPC client and server code, as well as the regular protocol buffer code for populating, serializing, and retrieving your message types. To learn more about protocol buffers, including how to install `protoc` with the gRPC plugin in your chosen language, see the [protocol buffers documentation](https://protobuf.dev/overview).

​	gRPC 使用带有特殊 gRPC 插件的 `protoc` 从您的 proto 文件生成代码：您将获得生成的 gRPC 客户端和服务器代码，以及用于填充、序列化和检索消息类型的常规协议缓冲区代码。要了解有关协议缓冲区的更多信息，包括如何在您选择的语言中安装带 gRPC 插件的 `protoc`，请参阅 [协议缓冲区文档](https://protobuf.dev/overview)。

## Protocol buffer versions

While [protocol buffers](https://protobuf.dev/overview) have been available to open source users for some time, most examples from this site use protocol buffers version 3 (proto3), which has a slightly simplified syntax, some useful new features, and supports more languages. Proto3 is currently available in Java, C++, Dart, Python, Objective-C, C#, a lite-runtime (Android Java), Ruby, and JavaScript from the [protocol buffers GitHub repo](https://github.com/google/protobuf/releases), as well as a Go language generator from the [golang/protobuf official package](https://pkg.go.dev/google.golang.org/protobuf), with more languages in development. You can find out more in the [proto3 language guide](https://protobuf.dev/programming-guides/proto3) and the [reference documentation](https://protobuf.dev/reference) available for each language. The reference documentation also includes a [formal specification](https://protobuf.dev/reference/protobuf/proto3-spec) for the `.proto` file format.

​	虽然 [协议缓冲区](https://protobuf.dev/overview) 已经向开源用户开放了一段时间，但本网站的大多数示例都使用协议缓冲区版本 3（proto3）。proto3 提供了一种稍微简化的语法、一些有用的新功能，并支持更多语言。proto3 当前可用于 Java、C++、Dart、Python、Objective-C、C#、精简运行时（Android Java）、Ruby 和 JavaScript（通过 [协议缓冲区 GitHub 仓库](https://github.com/google/protobuf/releases)），以及 Go 语言生成器（通过 [golang/protobuf 官方包](https://pkg.go.dev/google.golang.org/protobuf)），更多语言仍在开发中。您可以在 [proto3 语言指南](https://protobuf.dev/programming-guides/proto3) 和每种语言的 [参考文档](https://protobuf.dev/reference) 中找到更多信息。参考文档还包括 [.proto 文件格式的正式规范](https://protobuf.dev/reference/protobuf/proto3-spec)。

In general, while you can use proto2 (the current default protocol buffers version), we recommend that you use proto3 with gRPC as it lets you use the full range of gRPC-supported languages, as well as avoiding compatibility issues with proto2 clients talking to proto3 servers and vice versa.

​	一般来说，尽管您可以使用 proto2（当前的默认协议缓冲区版本），但我们建议您在 gRPC 中使用 proto3，因为它允许您使用 gRPC 支持的所有语言，并避免 proto2 客户端与 proto3 服务器（或反之）通信时的兼容性问题。
