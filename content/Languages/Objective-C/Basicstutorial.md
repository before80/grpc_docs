+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/objective-c/basics/](https://grpc.io/docs/languages/objective-c/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC in Objective-C.

​	Objective-C 中 gRPC 的基本教程简介。



This tutorial provides a basic Objective-C programmer’s introduction to working with gRPC.

​	本教程为 Objective-C 程序员提供了使用 gRPC 的基本入门知识。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a .proto file.
  在 .proto 文件中定义服务。
- Generate client code using the protocol buffer compiler.
  使用协议缓冲区编译器生成客户端代码。
- Use the Objective-C gRPC API to write a simple client for your service.
  使用 Objective-C gRPC API 为您的服务编写一个简单的客户端。

It assumes a passing familiarity with [protocol buffers](https://protobuf.dev/overview). Note that the example in this tutorial uses the proto3 version of the protocol buffers language: you can find out more in the [proto3 language guide](https://protobuf.dev/programming-guides/proto3) and the [Objective-C generated code guide](https://protobuf.dev/reference/objective-c/objective-c-generated).

​	它假定您对协议缓冲区有一定了解。请注意，本教程中的示例使用协议缓冲区语言的 proto3 版本：您可以在 proto3 语言指南和 Objective-C 生成的代码指南中了解更多信息。

### Why use gRPC? 为什么要使用 gRPC？

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	使用 gRPC，我们可以在 `.proto` 文件中定义一次服务，并在 gRPC 支持的任何语言中生成客户端和服务器，然后可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行这些客户端和服务器——gRPC 会为您处理不同语言和环境之间的所有通信复杂性。我们还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。

### Example code and setup 示例代码和设置

The example code for our tutorial is in [grpc/grpc/examples/objective-c/route_guide](https://github.com/grpc/grpc/tree/v1.60.0/examples/objective-c/route_guide). To download the example, clone the `grpc` repository by running the following commands:

​	本教程的示例代码位于 grpc/grpc/examples/objective-c/route_guide 中。要下载示例，请通过运行以下命令克隆 `grpc` 存储库：

```sh
$ git clone -b v1.60.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc
$ cd grpc
$ git submodule update --init
```

Then change your current directory to `examples/objective-c/route_guide`:

​	然后将当前目录更改为 `examples/objective-c/route_guide` ：

```sh
$ cd examples/objective-c/route_guide
```

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

You also should have [Cocoapods](https://cocoapods.org/#install) installed, as well as the relevant tools to generate the client library code (and a server in another language, for testing). You can obtain the latter by following [these setup instructions](https://github.com/grpc/homebrew-grpc).

​	您还应该安装 Cocoapods，以及生成客户端库代码（以及用于测试的另一种语言的服务器）的相关工具。您可以按照这些设置说明获取后者。

## Try it out! 试一试！

To try the sample app, we need a gRPC server running locally. Let’s compile and run, for example, the C++ server in this repository:

​	要试用示例应用，我们需要在本地运行一个 gRPC 服务器。例如，让我们编译并运行此存储库中的 C++ 服务器：

```sh
$ pushd ../../cpp/route_guide
$ make
$ ./route_guide_server &
$ popd
```

Now have Cocoapods generate and install the client library for our .proto files:

​	现在让 Cocoapods 为我们的 .proto 文件生成并安装客户端库：

```sh
$ pod install
```

(This might have to compile OpenSSL, which takes around 15 minutes if Cocoapods doesn’t have it yet on your computer’s cache).

​	（这可能必须编译 OpenSSL，如果 Cocoapods 尚未在您计算机的缓存中，则大约需要 15 分钟）。

Finally, open the XCode workspace created by Cocoapods, and run the app. You can check the calling code in `ViewControllers.m` and see the results in XCode’s log console.

​	最后，打开 Cocoapods 创建的 XCode 工作空间，并运行该应用。您可以在 `ViewControllers.m` 中检查调用代码，并在 XCode 的日志控制台中查看结果。

The next sections guide you step-by-step through how this proto service is defined, how to generate a client library from it, and how to create an app that uses that library.

​	接下来的部分将逐步指导您完成如何定义此 proto 服务、如何从中生成客户端库以及如何创建使用该库的应用。

### Defining the service 定义服务

First let’s look at how the service we’re using is defined. A gRPC *service* and its method *request* and *response* types using [protocol buffers](https://protobuf.dev/overview). You can see the complete .proto file for our example in [`examples/protos/route_guide.proto`](https://github.com/grpc/grpc/blob/v1.60.0/examples/protos/route_guide.proto).

​	首先，我们来看看我们正在使用的服务是如何定义的。gRPC 服务及其使用协议缓冲区的方法请求和响应类型。您可以在 `examples/protos/route_guide.proto` 中看到我们示例的完整 .proto 文件。

To define a service, you specify a named `service` in your .proto file:

​	要定义服务，您需要在 .proto 文件中指定一个命名的 `service` ：

```protobuf
service RouteGuide {
   ...
}
```

Then you define `rpc` methods inside your service definition, specifying their request and response types. Protocol buffers let you define four kinds of service method, all of which are used in the `RouteGuide` service:

​	然后，您可以在服务定义中定义 `rpc` 方法，指定它们各自的请求和响应类型。协议缓冲区允许您定义四种服务方法，所有这些方法都用于 `RouteGuide` 服务：

- A *simple RPC* where the client sends a request to the server and receives a response later, just like a normal remote procedure call.

  ​	一个简单的 RPC，其中客户端向服务器发送请求，稍后收到响应，就像一个普通的远程过程调用。

  ```protobuf
  // Obtains the feature at a given position.
  rpc GetFeature(Point) returns (Feature) {}
  ```

- A *response-streaming RPC* where the client sends a request to the server and gets back a stream of response messages. You specify a response-streaming method by placing the `stream` keyword before the *response* type.

  ​	一个响应流式 RPC，其中客户端向服务器发送请求并返回一个响应消息流。您可以通过在响应类型前放置 `stream` 关键字来指定响应流式方法。

  ```protobuf
  // Obtains the Features available within the given Rectangle.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field), as the rectangle may cover a large area and contain a
  // huge number of features.
  rpc ListFeatures(Rectangle) returns (stream Feature) {}
  ```

- A *request-streaming RPC* where the client sends a sequence of messages to the server. Once the client has finished writing the messages, it waits for the server to read them all and return its response. You specify a request-streaming method by placing the `stream` keyword before the *request* type.

  ​	客户端向服务器发送一系列消息的请求流式 RPC。一旦客户端完成编写消息，它将等待服务器读取所有消息并返回其响应。通过在请求类型前放置 `stream` 关键字来指定请求流式方法。

  ```protobuf
  // Accepts a stream of Points on a route being traversed, returning a
  // RouteSummary when traversal is completed.
  rpc RecordRoute(stream Point) returns (RouteSummary) {}
  ```

- A *bidirectional streaming RPC* where both sides send a sequence of messages to the other. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved. You specify this type of method by placing the `stream` keyword before both the request and the response.

  ​	双方都向对方发送一系列消息的双向流式 RPC。两个流独立运行，因此客户端和服务器可以按任何顺序进行读写：例如，服务器可以等到收到所有客户端消息后再编写其响应，或者可以交替读取消息然后编写消息，或其他一些读写组合。每个流中的消息顺序保持不变。通过在请求和响应前放置 `stream` 关键字来指定此类型的函数。

  ```protobuf
  // Accepts a stream of RouteNotes sent while a route is being traversed,
  // while receiving other RouteNotes (e.g. from other users).
  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
  ```

Our `.proto` file also contains protocol buffer message type definitions for all the request and response types used in our service methods - for example, here’s the `Point` message type:

​	我们的 `.proto` 文件还包含用于我们服务函数中的所有请求和响应类型的协议缓冲区消息类型定义 - 例如，以下是 `Point` 消息类型：

```protobuf
// Points are represented as latitude-longitude pairs in the E7 representation
// (degrees multiplied by 10**7 and rounded to the nearest integer).
// Latitudes should be in the range +/- 90 degrees and longitude should be in
// the range +/- 180 degrees (inclusive).
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}
```

You can specify a prefix to be used for your generated classes by adding the `objc_class_prefix` option at the top of the file. For example:

​	您可以通过在文件顶部添加 `objc_class_prefix` 选项来指定要用于生成的类的前缀。例如：

```protobuf
option objc_class_prefix = "RTG";
```

### Generating client code 生成客户端代码

Next we need to generate the gRPC client interfaces from our .proto service definition. We do this using the protocol buffer compiler (`protoc`) with a special gRPC Objective-C plugin.

​	接下来，我们需要从 .proto 服务定义中生成 gRPC 客户端接口。我们使用协议缓冲区编译器 ( `protoc` ) 和特殊的 gRPC Objective-C 插件来执行此操作。

For simplicity, we’ve provided a [Podspec file](https://github.com/grpc/grpc/blob/v1.60.0/examples/objective-c/route_guide/RouteGuide.podspec) that runs `protoc` for you with the appropriate plugin, input, and output, and describes how to compile the generated files. You just need to run in this directory (`examples/objective-c/route_guide`):

​	为了简单起见，我们提供了一个 Podspec 文件，它使用适当的插件、输入和输出为您运行 `protoc` ，并说明如何编译生成的文件。您只需在此目录中运行 ( `examples/objective-c/route_guide` ):

```sh
$ pod install
```

which, before installing the generated library in the XCode project of this sample, runs:

​	在 XCode 项目中安装生成的库之前，运行：

```sh
$ protoc -I ../../protos --objc_out=Pods/RouteGuide --objcgrpc_out=Pods/RouteGuide ../../protos/route_guide.proto
```

Running this command generates the following files under `Pods/RouteGuide/`:

​	运行此命令将在 `Pods/RouteGuide/` 下生成以下文件：

- `RouteGuide.pbobjc.h`, the header which declares your generated message classes.
  `RouteGuide.pbobjc.h` ，声明生成的 message 类别的头文件。
- `RouteGuide.pbobjc.m`, which contains the implementation of your message classes.
  `RouteGuide.pbobjc.m` ，包含 message 类别的实现。
- `RouteGuide.pbrpc.h`, the header which declares your generated service classes.
  `RouteGuide.pbrpc.h` ，声明生成的 service 类别的头文件。
- `RouteGuide.pbrpc.m`, which contains the implementation of your service classes.
  `RouteGuide.pbrpc.m` ，包含 service 类别的实现。

These contain: 
​	它们包含：

- All the protocol buffer code to populate, serialize, and retrieve our request and response message types.
  用于填充、序列化和检索请求和响应消息类型的所有协议缓冲区代码。
- A class called `RTGRouteGuide` that lets clients call the methods defined in the `RouteGuide` service.
  一个名为 `RTGRouteGuide` 的类，允许客户端调用 `RouteGuide` 服务中定义的方法。

You can also use the provided Podspec file to generate client code from any other proto service definition; just replace the name (matching the file name), version, and other metadata.

​	您还可以使用提供的 Podspec 文件从任何其他 proto 服务定义生成客户端代码；只需替换名称（与文件名匹配）、版本和其他元数据。

### Creating the client application 创建客户端应用程序

In this section, we’ll look at creating an Objective-C client for our `RouteGuide` service. You can see our complete example client code in [examples/objective-c/route_guide/ViewControllers.m](https://github.com/grpc/grpc/blob/v1.60.0/examples/objective-c/route_guide/ViewControllers.m).

​	在本节中，我们将研究为 `RouteGuide` 服务创建 Objective-C 客户端。您可以在 examples/objective-c/route_guide/ViewControllers.m 中看到我们完整的示例客户端代码。

#### Note 注意

In your apps, for maintainability and readability reasons, you shouldn’t put all of your view controllers in a single file; it’s done here only to simplify the learning process).
在您的应用中，出于可维护性和可读性原因，您不应该将所有视图控制器都放在一个文件中；这里这样做只是为了简化学习过程）。

#### Constructing a service object 构建服务对象

To call service methods, we first need to create a service object, an instance of the generated `RTGRouteGuide` class. The designated initializer of the class expects a `NSString *` with the server address and port we want to connect to:

​	要调用服务方法，我们首先需要创建一个服务对象，即生成的 `RTGRouteGuide` 类的实例。该类的指定初始化程序需要一个 `NSString *` ，其中包含我们要连接到的服务器地址和端口：

```objective-c
#import <GRPCClient/GRPCCall+Tests.h>
#import <RouteGuide/RouteGuide.pbrpc.h>
#import <GRPCClient/GRPCTransport.h>

static NSString * const kHostAddress = @"localhost:50051";
...
GRPCMutableCallOptions *options = [[GRPCMutableCallOptions alloc] init];
options.transport = GRPCDefaultTransportImplList.core_insecure;

RTGRouteGuide *service = [[RTGRouteGuide alloc] initWithHost:kHostAddress callOptions:options];
```

Notice that we our service is constructed with an insecure transport. This is because the server we will be using to test our client doesn’t use [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security). This is fine because it will be running locally on our development machine. The most common case, though, is connecting with a gRPC server on the internet, running gRPC over TLS. For that case, the setting the option `options.transport` isn’t needed because gRPC will use a secure TLS transport by default.

​	请注意，我们的服务是使用不安全的传输构建的。这是因为我们将用于测试客户端的服务器不使用 TLS。这很好，因为它将在我们的开发机器上本地运行。然而，最常见的情况是通过互联网连接到 gRPC 服务器，通过 TLS 运行 gRPC。对于这种情况，不需要设置选项 `options.transport` ，因为 gRPC 默认会使用安全的 TLS 传输。

#### Calling service methods 调用服务方法

Now let’s look at how we call our service methods. As you will see, all these methods are asynchronous, so you can call them from the main thread of your app without worrying about freezing your UI or the OS killing your app.

​	现在让我们看看如何调用我们的服务方法。您将看到，所有这些方法都是异步的，因此您可以从应用程序的主线程调用它们，而无需担心冻结 UI 或操作系统终止您的应用程序。

##### Simple RPC 简单 RPC

Calling the simple RPC `GetFeature` is as straightforward as calling any other asynchronous method on Cocoa.

​	调用简单的 RPC `GetFeature` 与在 Cocoa 上调用任何其他异步方法一样简单。

```objective-c
RTGPoint *point = [RTGPoint message];
point.latitude = 40E7;
point.longitude = -74E7;

GRPCUnaryResponseHandler *handler =
    [[GRPCUnaryResponseHandler alloc] initWithResponseHandler:
        ^(RTGFeature *response, NSError *error) {
          if (response) {
            // Successful response received
          } else {
            // RPC error
          }
        }
                                        responseDispatchQueue:nil];

[[service getFeatureWithMessage:point responseHandler:handler callOptions:nil] start];
```

As you can see, we create and populate a request protocol buffer object (in our case `RTGPoint`). Then, we call the method on the service object, passing it the request, and a block to handle the response (or any RPC error). If the RPC finishes successfully, the handler block is called with a `nil` error argument, and we can read the response information from the server from the response argument. If, instead, some RPC error happens, the handler block is called with a `nil` response argument, and we can read the details of the problem from the error argument.

​	如您所见，我们创建并填充了一个请求协议缓冲区对象（在本例中为 `RTGPoint` ）。然后，我们在服务对象上调用该方法，将请求和一个用于处理响应（或任何 RPC 错误）的块传递给它。如果 RPC 成功完成，则使用 `nil` 错误参数调用处理程序块，我们可以从响应参数中读取服务器的响应信息。如果发生某些 RPC 错误，则使用 `nil` 响应参数调用处理程序块，我们可以从错误参数中读取问题的详细信息。

##### Streaming RPCs 流式 RPC

Now let’s look at our streaming methods. Here’s where we call the response-streaming method `ListFeatures`, which results in our client app receiving a stream of geographical `RTGFeature`s:

​	现在让我们看看我们的流方法。我们在这里调用响应流方法 `ListFeatures` ，这会导致我们的客户端应用程序接收地理 `RTGFeature` 流：

```objective-c
- (void)didReceiveProtoMessage(GPBMessage *)message {
  if (message) {
    NSLog(@"Found feature at %@ called %@.", response.location, response.name);
  }
}

- (void)didCloseWithTrailingMetadata:(NSDictionary *)trailingMetadata error:(NSError *)error {
  if (error) {
    NSLog(@"RPC error: %@", error);
  }
}

- (void)execRequest {
  ...
  [[service listFeaturesWithMessage:rectangle responseHandler:self callOptions:nil] start];
}
```

Notice that instead of providing a response handler object, the view controller object itself handles the responses. The method `didReceiveProtoMessage:` is called when there’s a message received; it can be called any number of times. The method `didCloseWithTrailingMetadata:` is called when the call is complete and the gRPC status is received from the server (or when there’s any error happens during the call).

​	请注意，视图控制器对象本身处理响应，而不是提供响应处理程序对象。当收到消息时，将调用方法 `didReceiveProtoMessage:` ；可以多次调用它。当调用完成并且从服务器收到 gRPC 状态（或在调用期间发生任何错误时），将调用方法 `didCloseWithTrailingMetadata:` 。

The request-streaming method `RecordRoute` expects a stream of `RTGPoint`s from the cient. This stream can be written to the gRPC call object after the call starts.

​	请求流方法 `RecordRoute` 期望从客户端获得 `RTGPoint` 流。可以在调用开始后将此流写入 gRPC 调用对象。

```objective-c
RTGPoint *point1 = [RTGPoint message];
point.latitude = 40E7;
point.longitude = -74E7;

RTGPoint *point2 = [RTGPoint message];
point.latitude = 40E7;
point.longitude = -74E7;

GRPCUnaryResponseHandler *handler =
    [[GRPCUnaryResponseHandler alloc] initWithResponseHandler:
        ^(RTGRouteSummary *response, NSError *error) {
            if (response) {
              NSLog(@"Finished trip with %i points", response.pointCount);
              NSLog(@"Passed %i features", response.featureCount);
              NSLog(@"Travelled %i meters", response.distance);
              NSLog(@"It took %i seconds", response.elapsedTime);
            } else {
              NSLog(@"RPC error: %@", error);
            }
        }
                                        responseDispatchQueue:nil];
GRPCStreamingProtoCall *call =
    [service recordRouteWithResponseHandler:handler callOptions:nil];
[call start];
[call writeMessage:point1];
[call writeMessage:point2];
[call finish];
```

Note that since the gRPC call object does not know the end of the request stream, users must invoke `finish:` method when the request stream is complete.

​	请注意，由于 gRPC 调用对象不知道请求流的结束，因此用户必须在请求流完成后调用 `finish:` 方法。

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`. The way to call a bidirectional streaming RPC is just a combination of how to call request-streaming RPCs and response-streaming RPCs.

​	最后，我们来看一下双向流式 RPC `RouteChat()` 。调用双向流式 RPC 的方法只是调用请求流式 RPC 和响应流式 RPC 的方法的组合。

```objective-c
- (void)didReceiveProtoMessage(GPBMessage *)message {
  RTGRouteNote *note = (RTGRouteNote *)message;
  if (note) {
    NSLog(@"Got message %@ at %@", note.message, note.location);
  }
}

- (void)didCloseWithTrailingMetadata:(NSDictionary *)trailingMetadata error:(NSError *)error {
  if (error) {
    NSLog(@"RPC error: %@", error);
  } else {
    NSLog(@"Chat ended.");
  }
}

- (void)execRequest {
  ...
  GRPCStreamingProtoCall *call =
      [service routeChatWithResponseHandler:self callOptions:nil];
  [call start];
  [call writeMessage:note1];
  ...
  [call writeMessage:noteN];
  [call finish];
}
```
