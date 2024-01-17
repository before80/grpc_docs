+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/ruby/basics/](https://grpc.io/docs/languages/ruby/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC in Ruby.



This tutorial provides a basic Ruby programmer’s introduction to working with gRPC.

​	本教程为 Ruby 程序员提供了使用 gRPC 的基本入门知识。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a .proto file.
  在 .proto 文件中定义服务。
- Generate server and client code using the protocol buffer compiler.
  使用协议缓冲区编译器生成服务器和客户端代码。
- Use the Ruby gRPC API to write a simple client and server for your service.
  使用 Ruby gRPC API 为您的服务编写一个简单的客户端和服务器。

It assumes that you have read the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and are familiar with [protocol buffers](https://protobuf.dev/overview). Note that the example in this tutorial uses the proto3 version of the protocol buffers language: you can find out more in the [proto3 language guide](https://protobuf.dev/programming-guides/proto3).

​	它假定您已阅读 gRPC 简介并且熟悉协议缓冲区。请注意，本教程中的示例使用协议缓冲区语言的 proto3 版本：您可以在 proto3 语言指南中了解更多信息。

### Why use gRPC? 为什么要使用 gRPC？

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	使用 gRPC，我们可以在 `.proto` 文件中定义一次服务，并在 gRPC 支持的任何语言中生成客户端和服务器，然后可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行这些客户端和服务器——gRPC 会为您处理不同语言和环境之间的所有通信复杂性。我们还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。

### Example code and setup 示例代码和设置

The example code for our tutorial is in [grpc/grpc/examples/ruby/route_guide](https://github.com/grpc/grpc/tree/v1.60.0/examples/ruby/route_guide). To download the example, clone the `grpc` repository by running the following command:

​	我们教程的示例代码位于 grpc/grpc/examples/ruby/route_guide。要下载示例，请通过运行以下命令克隆 `grpc` 存储库：

```sh
$ git clone -b v1.60.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc
$ cd grpc
```

Then change your current directory to `examples/ruby/route_guide`:

​	然后将当前目录更改为 `examples/ruby/route_guide` ：

```sh
$ cd examples/ruby/route_guide
```

You also should have the relevant tools installed to generate the server and client interface code - if you don’t already, follow the setup instructions in [Quick start]({{< ref "/Languages/Ruby/Quickstart" >}}).

​	您还应该安装相关工具来生成服务器和客户端接口代码 - 如果您还没有安装，请按照快速入门中的设置说明进行操作。

### Defining the service 定义服务

Our first step (as you’ll know from the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}})) is to define the gRPC *service* and the method *request* and *response* types using [protocol buffers](https://protobuf.dev/overview). You can see the complete .proto file in [`examples/protos/route_guide.proto`](https://github.com/grpc/grpc/blob/v1.60.0/examples/protos/route_guide.proto).

​	我们的第一步（如您从 gRPC 简介中了解到的）是使用协议缓冲区定义 gRPC 服务以及方法请求和响应类型。您可以在 `examples/protos/route_guide.proto` 中看到完整的 .proto 文件。

To define a service, you specify a named `service` in your .proto file:

​	要定义服务，您需要在 .proto 文件中指定一个命名的 `service` ：

```protobuf
service RouteGuide {
   ...
}
```

Then you define `rpc` methods inside your service definition, specifying their request and response types. gRPC lets you define four kinds of service method, all of which are used in the `RouteGuide` service:

​	然后在服务定义中定义 `rpc` 方法，指定它们的请求和响应类型。gRPC 允许您定义四种服务方法，所有这些方法都用于 `RouteGuide` 服务：

- A *simple RPC* where the client sends a request to the server using the stub and waits for a response to come back, just like a normal function call.

  ​	一个简单的 RPC，其中客户端使用存根向服务器发送请求并等待响应返回，就像一个普通的函数调用一样。

  ```protobuf
  // Obtains the feature at a given position.
  rpc GetFeature(Point) returns (Feature) {}
  ```

- A *server-side streaming RPC* where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. As you can see in our example, you specify a server-side streaming method by placing the `stream` keyword before the *response* type.

  ​	一个服务器端流式 RPC，其中客户端向服务器发送请求并获取一个流来读取一系列消息。客户端从返回的流中读取，直到没有更多消息。如您在我们的示例中所见，您可以通过在响应类型之前放置 `stream` 关键字来指定服务器端流式方法。

  ```protobuf
  // Obtains the Features available within the given Rectangle.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field), as the rectangle may cover a large area and contain a
  // huge number of features.
  rpc ListFeatures(Rectangle) returns (stream Feature) {}
  ```

- A *client-side streaming RPC* where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them all and return its response. You specify a client-side streaming method by placing the `stream` keyword before the *request* type.

  ​	一个客户端流式 RPC，其中客户端编写一系列消息并使用提供的流将它们发送到服务器。一旦客户端完成编写消息，它就会等待服务器读取所有消息并返回其响应。您可以通过在请求类型之前放置 `stream` 关键字来指定客户端流式方法。

  ```protobuf
  // Accepts a stream of Points on a route being traversed, returning a
  // RouteSummary when traversal is completed.
  rpc RecordRoute(stream Point) returns (RouteSummary) {}
  ```

- A *bidirectional streaming RPC* where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved. You specify this type of method by placing the `stream` keyword before both the request and the response.

  ​	双方使用读写流发送一系列消息的双向流式 RPC。两个流独立运行，因此客户端和服务器可以按照他们喜欢的任何顺序进行读写：例如，服务器可以等到收到所有客户端消息后再写出其响应，或者可以交替读取消息然后写出消息，或者其他一些读写组合。每个流中的消息顺序都将保留。通过在请求和响应之前放置 `stream` 关键字来指定此类型的函数。

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

### Generating client and server code 生成客户端和服务器代码

Next we need to generate the gRPC client and server interfaces from our .proto service definition. We do this using the protocol buffer compiler `protoc` with a special gRPC Ruby plugin.

​	接下来，我们需要从我们的 .proto 服务定义中生成 gRPC 客户端和服务器接口。我们使用协议缓冲区编译器 `protoc` 和特殊的 gRPC Ruby 插件来执行此操作。

If you want to run this yourself, make sure you have installed [gRPC]({{< ref "/Languages/Ruby/Quickstart#grpc" >}}) and [protoc]({{< ref "/Languages/Ruby/Quickstart#grpc-tools" >}}).

​	如果您想自己运行此操作，请确保已安装 gRPC 和 protoc。

Once that’s done, the following command can be used to generate the ruby code.

​	完成后，可以使用以下命令生成 Ruby 代码。

```sh
$ grpc_tools_ruby_protoc -I ../../protos --ruby_out=../lib --grpc_out=../lib ../../protos/route_guide.proto
```

Running this command regenerates the following files in the lib directory:

​	运行此命令将在 lib 目录中重新生成以下文件：

- ```
  lib/route_guide.pb
  ```

   

  defines a module

   

  ```
  Examples::RouteGuide
  ```

  
  `lib/route_guide.pb` 定义了一个模块 `Examples::RouteGuide`

  - This contain all the protocol buffer code to populate, serialize, and retrieve our request and response message types
    其中包含所有协议缓冲区代码，用于填充、序列化和检索我们的请求和响应消息类型

- ```
  lib/route_guide_services.pb
  ```

  , extends

   

  ```
  Examples::RouteGuide
  ```

   

  with stub and service classes

  
  `lib/route_guide_services.pb` ，使用存根和服务类扩展 `Examples::RouteGuide`

  - a class `Service` for use as a base class when defining RouteGuide service implementations
    一个类 `Service` ，用作定义 RouteGuide 服务实现时的基类
  - a class `Stub` that can be used to access remote RouteGuide instances
    一个可用于访问远程 RouteGuide 实例的类 `Stub` gRPC 在 Ruby 中的基本教程介绍。

### Creating the server 创建服务器

First let’s look at how we create a `RouteGuide` server. If you’re only interested in creating gRPC clients, you can skip this section and go straight to [Creating the client]({{< ref "/Languages/Ruby/Basicstutorial#client" >}}) (though you might find it interesting anyway!).

​	首先，我们来看看如何创建一个 `RouteGuide` 服务器。如果您只对创建 gRPC 客户端感兴趣，可以跳过本部分，直接转到创建客户端（不过您可能还是会觉得它很有趣！）。

There are two parts to making our `RouteGuide` service do its job:

​	让我们的 `RouteGuide` 服务发挥作用有两个部分：

- Implementing the service interface generated from our service definition: doing the actual “work” of our service.
  实现从服务定义中生成的服务接口：执行服务的实际“工作”。
- Running a gRPC server to listen for requests from clients and return the service responses.
  运行 gRPC 服务器以侦听来自客户端的请求并返回服务响应。

You can find our example `RouteGuide` server in [examples/ruby/route_guide/route_guide_server.rb](https://github.com/grpc/grpc/blob/v1.60.0/examples/ruby/route_guide/route_guide_server.rb). Let’s take a closer look at how it works.
您可以在 examples/ruby/route_guide/route_guide_server.rb 中找到我们的示例 `RouteGuide` 服务器。让我们仔细看看它是如何工作的。

#### Implementing RouteGuide 实现 RouteGuide

As you can see, our server has a `ServerImpl` class that extends the generated `RouteGuide::Service`:
如您所见，我们的服务器有一个 `ServerImpl` 类，它扩展了生成的 `RouteGuide::Service` ：

```ruby
# ServerImpl provides an implementation of the RouteGuide service.
class ServerImpl < RouteGuide::Service
```

`ServerImpl` implements all our service methods. Let’s look at the simplest type first, `GetFeature`, which just gets a `Point` from the client and returns the corresponding feature information from its database in a `Feature`.
`ServerImpl` 实现我们所有的服务方法。我们先来看最简单的类型 `GetFeature` ，它只是从客户端获取 `Point` ，并从其数据库中的 `Feature` 中返回相应的要素信息。

```ruby
def get_feature(point, _call)
  name = @feature_db[{
    'longitude' => point.longitude,
    'latitude' => point.latitude }] || ''
  Feature.new(location: point, name: name)
end
```

The method is passed a _call for the RPC, the client’s `Point` protocol buffer request, and returns a `Feature` protocol buffer. In the method we create the `Feature` with the appropriate information, and then `return` it.
该方法传递一个 RPC 的 _call、客户端的 `Point` 协议缓冲区请求，并返回一个 `Feature` 协议缓冲区。在该方法中，我们使用适当的信息创建 `Feature` ，然后 `return` 它。

Now let’s look at something a bit more complicated - a streaming RPC. `ListFeatures` is a server-side streaming RPC, so we need to send back multiple `Feature`s to our client.
现在，我们来看一些更复杂的内容 - 流式 RPC。 `ListFeatures` 是服务器端流式 RPC，因此我们需要向我们的客户端发送多个 `Feature` 。

```ruby
# in ServerImpl

  def list_features(rectangle, _call)
    RectangleEnum.new(@feature_db, rectangle).each
  end
```

As you can see, here the request object is a `Rectangle` in which our client wants to find `Feature`s, but instead of returning a simple response we need to return an [Enumerator](https://ruby-doc.org//core-2.2.0/Enumerator.html) that yields the responses. In the method, we use a helper class `RectangleEnum`, to act as an Enumerator implementation.
如您所见，这里的请求对象是一个 `Rectangle` ，其中我们的客户端想要查找 `Feature` ，但我们不需要返回一个简单的响应，而是需要返回一个产生响应的枚举器。在该方法中，我们使用一个帮助类 `RectangleEnum` ，作为枚举器实现。

Similarly, the client-side streaming method `record_route` uses an [Enumerable](https://ruby-doc.org//core-2.2.0/Enumerable.html), but here it’s obtained from the call object, which we’ve ignored in the earlier examples. `call.each_remote_read` yields each message sent by the client in turn.
类似地，客户端流方法 `record_route` 使用一个 Enumerable，但这里它是从 call 对象获取的，我们在前面的示例中忽略了它。 `call.each_remote_read` 依次产生客户端发送的每条消息。

```ruby
call.each_remote_read do |point|
  ...
end
```

Finally, let’s look at our bidirectional streaming RPC `route_chat`.
最后，我们来看一下我们的双向流 RPC `route_chat` 。

```ruby
def route_chat(notes)
  RouteChatEnumerator.new(notes, @received_notes).each_item
end
```

Here the method receives an [Enumerable](https://ruby-doc.org//core-2.2.0/Enumerable.html), but also returns an [Enumerator](https://ruby-doc.org//core-2.2.0/Enumerator.html) that yields the responses. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.
在此方法接收一个 Enumerable，但也会返回一个生成响应的 Enumerator。尽管每一方总是按写入顺序获取另一方的消息，但客户端和服务器都可以按任何顺序进行读写——流完全独立地运行。

#### Starting the server 启动服务器

Once we’ve implemented all our methods, we also need to start up a gRPC server so that clients can actually use our service. The following snippet shows how we do this for our `RouteGuide` service:
实现所有方法后，我们还需要启动一个 gRPC 服务器，以便客户端实际使用我们的服务。以下代码段展示了我们如何为 `RouteGuide` 服务执行此操作：

```ruby
port = '0.0.0.0:50051'
s = GRPC::RpcServer.new
s.add_http2_port(port, :this_port_is_insecure)
GRPC.logger.info("... running insecurely on #{port}")
s.handle(ServerImpl.new(feature_db))
# Runs the server with SIGHUP, SIGINT and SIGQUIT signal handlers to
#   gracefully shutdown.
# User could also choose to run server via call to run_till_terminated
s.run_till_terminated_or_interrupted([1, 'int', 'SIGQUIT'])
```

As you can see, we build and start our server using a `GRPC::RpcServer`. To do this, we:
如您所见，我们使用 `GRPC::RpcServer` 构建并启动服务器。为此，我们：

1. Create an instance of our service implementation class `ServerImpl`.
   创建一个服务实现类 `ServerImpl` 的实例。
2. Specify the address and port we want to use to listen for client requests using the builder’s `add_http2_port` method.
   使用构建器的 `add_http2_port` 方法指定我们想要用于侦听客户端请求的地址和端口。
3. Register our service implementation with the `GRPC::RpcServer`.
   使用 `GRPC::RpcServer` 注册我们的服务实现。
4. Call `run` on the`GRPC::RpcServer` to create and start an RPC server for our service.
   在 `GRPC::RpcServer` 上调用 `run` 以创建和启动我们服务的 RPC 服务器。

### Creating the client 创建客户端

In this section, we’ll look at creating a Ruby client for our `RouteGuide` service. You can see our complete example client code in [examples/ruby/route_guide/route_guide_client.rb](https://github.com/grpc/grpc/blob/v1.60.0/examples/ruby/route_guide/route_guide_client.rb).
在本节中，我们将介绍如何为我们的 `RouteGuide` 服务创建 Ruby 客户端。您可以在 examples/ruby/route_guide/route_guide_client.rb 中看到我们完整的示例客户端代码。

#### Creating a stub 创建存根

To call service methods, we first need to create a *stub*.
要调用服务方法，我们首先需要创建一个存根。

We use the `Stub` class of the `RouteGuide` module generated from our .proto.
我们使用从 .proto 生成的 `RouteGuide` 模块的 `Stub` 类。

```ruby
stub = RouteGuide::Stub.new('localhost:50051')
```

#### Calling service methods 调用服务方法

Now let’s look at how we call our service methods. Note that the gRPC Ruby only provides *blocking/synchronous* versions of each method: this means that the RPC call waits for the server to respond, and will either return a response or raise an exception.
现在让我们看看如何调用我们的服务方法。请注意，gRPC Ruby 仅提供每个方法的阻塞/同步版本：这意味着 RPC 调用会等待服务器响应，并且会返回响应或引发异常。

##### Simple RPC 简单 RPC

Calling the simple RPC `GetFeature` is nearly as straightforward as calling a local method.
调用简单 RPC `GetFeature` 几乎与调用本地方法一样简单。

```ruby
GET_FEATURE_POINTS = [
  Point.new(latitude:  409_146_138, longitude: -746_188_906),
  Point.new(latitude:  0, longitude: 0)
]
..
  GET_FEATURE_POINTS.each do |pt|
    resp = stub.get_feature(pt)
	...
    p "- found '#{resp.name}' at #{pt.inspect}"
  end
```

As you can see, we create and populate a request protocol buffer object (in our case `Point`), and create a response protocol buffer object for the server to fill in. Finally, we call the method on the stub, passing it the context, request, and response. If the method returns `OK`, then we can read the response information from the server from our response object.
如您所见，我们创建并填充请求协议缓冲区对象（在本例中为 `Point` ），并为服务器创建响应协议缓冲区对象以供其填写。最后，我们在存根上调用该方法，将上下文、请求和响应传递给它。如果该方法返回 `OK` ，那么我们就可以从响应对象中读取服务器的响应信息。

##### Streaming RPCs 流式 RPC

Now let’s look at our streaming methods. If you’ve already read [Creating the server]({{< ref "/Languages/Ruby/Basicstutorial#server" >}}) some of this may look very familiar - streaming RPCs are implemented in a similar way on both sides. Here’s where we call the server-side streaming method `list_features`, which returns an `Enumerable` of `Features`.
现在让我们看看我们的流方法。如果您已经阅读了创建服务器，其中一些内容可能看起来非常熟悉 - 流 RPC 在双方以类似的方式实现。我们在此调用服务器端流方法 `list_features` ，它返回 `Features` 的 `Enumerable` 。

```ruby
resps = stub.list_features(LIST_FEATURES_RECT)
resps.each do |r|
  p "- found '#{r.name}' at #{r.location.inspect}"
end
```

Non-blocking usage of the RPC stream can be achieved with multiple threads and the `return_op: true` flag. When passing the `return_op: true` flag, the execution of the RPC is deferred and an `Operation` object is returned. The RPC can then be executed in another thread by calling the operation `execute` function. The main thread can utilize contextual methods and getters such as `status`, `cancelled?`, and `cancel` to manage the RPC. This can be useful for persistent or long running RPC sessions that would block the main thread for an unacceptable period of time.
通过多个线程和 `return_op: true` 标志可以实现 RPC 流的非阻塞使用。传递 `return_op: true` 标志时，RPC 的执行被延迟，并返回一个 `Operation` 对象。然后可以通过调用操作 `execute` 函数在另一个线程中执行 RPC。主线程可以利用上下文方法和 getter（例如 `status` 、 `cancelled?` 和 `cancel` ）来管理 RPC。这对于会使主线程在不可接受的时间段内阻塞的持久或长时间运行的 RPC 会话非常有用。

```ruby
op = stub.list_features(LIST_FEATURES_RECT, return_op: true)
Thread.new do 
  resps = op.execute
  resps.each do |r|
    p "- found '#{r.name}' at #{r.location.inspect}"
  end
rescue GRPC::Cancelled => e
  p "operation cancel called - #{e}"
end

# controls for the operation
op.status
op.cancelled?
op.cancel # attempts to cancel the RPC with a GRPC::Cancelled status; there's a fundamental race condition where cancelling the RPC can race against RPC termination for a different reason - invoking `cancel` doesn't necessarily guarantee a `Cancelled` status
```

The client-side streaming method `record_route` is similar, except there we pass the server an `Enumerable`.
客户端流方法 `record_route` 类似，只是我们向服务器传递 `Enumerable` 。

```ruby
...
reqs = RandomRoute.new(features, points_on_route)
resp = stub.record_route(reqs.each)
...
```

Finally, let’s look at our bidirectional streaming RPC `route_chat`. In this case, we pass `Enumerable` to the method and get back an `Enumerable`.
最后，我们来看看双向流 RPC `route_chat` 。在这种情况下，我们将 `Enumerable` 传递给方法并获取 `Enumerable` 。

```ruby
sleeping_enumerator = SleepingEnumerator.new(ROUTE_CHAT_NOTES, 1)
stub.route_chat(sleeping_enumerator.each_item) { |r| p "received #{r.inspect}" }
```

Although it’s not shown well by this example, each enumerable is independent of the other - both the client and server can read and write in any order — the streams operate completely independently.
虽然此示例没有很好地展示这一点，但每个可枚举对象都是独立的——客户端和服务器都可以按任何顺序进行读写——流完全独立地运行。

### Try it out! 试一试！

Work from the example directory:
从示例目录中工作：

```sh
$ cd examples/ruby
```

Build the client and server:
构建客户端和服务器：

```sh
$ gem install bundler && bundle install
```

Run the server: 运行服务器：

```sh
$ bundle exec route_guide/route_guide_server.rb ../python/route_guide/route_guide_db.json
```

#### Note 注意

The `route_guide_db.json` file is actually language-agnostic, it happens to be located in the `python` folder.
`route_guide_db.json` 文件实际上与语言无关，它恰好位于 `python` 文件夹中。

From a different terminal, run the client:
在其他终端中，运行客户端：

```sh
$ bundle exec route_guide/route_guide_client.rb ../python/route_guide/route_guide_db.json
```
