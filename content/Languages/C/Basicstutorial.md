+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/cpp/basics/](https://grpc.io/docs/languages/cpp/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC in C++.

​	C++ 中 gRPC 的基本教程介绍。



This tutorial provides a basic C++ programmer’s introduction to working with gRPC.

​	本教程为 C++ 程序员提供了使用 gRPC 的基本介绍。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a `.proto` file.
  在 `.proto` 文件中定义服务。
- Generate server and client code using the protocol buffer compiler.
  使用协议缓冲区编译器生成服务器和客户端代码。
- Use the C++ gRPC API to write a simple client and server for your service.
  使用 C++ gRPC API 为您的服务编写简单的客户端和服务器。

It assumes that you have read the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and are familiar with [protocol buffers](https://protobuf.dev/overview). Note that the example in this tutorial uses the proto3 version of the protocol buffers language: you can find out more in the [proto3 language guide](https://protobuf.dev/programming-guides/proto3) and [C++ generated code guide](https://protobuf.dev/reference/cpp/cpp-generated).

​	它假定您已阅读 gRPC 简介并且熟悉协议缓冲区。请注意，本教程中的示例使用协议缓冲区语言的 proto3 版本：您可以在 proto3 语言指南和 C++ 生成的代码指南中了解更多信息。

### Why use gRPC? 为什么要使用 gRPC？

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	使用 gRPC，我们可以在 `.proto` 文件中定义一次服务，并在 gRPC 支持的任何语言中生成客户端和服务器，然后可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行这些客户端和服务器——gRPC 会为您处理不同语言和环境之间的所有通信复杂性。我们还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。

### Example code and setup 示例代码和设置

The example code is part of the `grpc` repo under [examples/cpp/route_guide](https://github.com/grpc/grpc/tree/v1.60.0/examples/cpp/route_guide). Get the example code and build gRPC:

​	示例代码是 `grpc` 存储库的一部分，位于 examples/cpp/route_guide 下。获取示例代码并构建 gRPC：

1. Follow the Quick start instructions to [build and locally install gRPC from source]({{< ref "/Languages/C/Quickstart#install-grpc" >}}).

   ​	按照快速入门说明从源代码构建并本地安装 gRPC。

2. From the repo folder, change to the route guide example directory:

   ​	从存储库文件夹切换到路线指南示例目录：

   ```sh
   $ cd examples/cpp/route_guide
   ```

3. Run `cmake` 
   ​	运行 `cmake`

   ```sh
   $ mkdir -p cmake/build
   $ cd cmake/build
   $ cmake -DCMAKE_PREFIX_PATH=$MY_INSTALL_DIR ../..
   ```

### Defining the service 定义服务

Our first step (as you’ll know from the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}})) is to define the gRPC *service* and the method *request* and *response* types using [protocol buffers](https://protobuf.dev/overview). You can see the complete `.proto` file in [`examples/protos/route_guide.proto`](https://github.com/grpc/grpc/blob/v1.60.0/examples/protos/route_guide.proto).

​	我们的第一步（您会从 gRPC 简介中了解到）是使用协议缓冲区定义 gRPC 服务以及方法请求和响应类型。您可以在 `examples/protos/route_guide.proto` 中看到完整的 `.proto` 文件。

To define a service, you specify a named `service` in your `.proto` file:

​	要定义服务，您需要在 `.proto` 文件中指定一个命名的 `service` ：

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

Next we need to generate the gRPC client and server interfaces from our `.proto` service definition. We do this using the protocol buffer compiler `protoc` with a special gRPC C++ plugin.

​	接下来，我们需要从我们的 `.proto` 服务定义中生成 gRPC 客户端和服务器接口。我们使用协议缓冲区编译器 `protoc` 和特殊的 gRPC C++ 插件来执行此操作。

For simplicity, we’ve provided a [CMakeLists.txt](https://github.com/grpc/grpc/blob/v1.60.0/examples/cpp/route_guide/CMakeLists.txt) that runs `protoc` for you with the appropriate plugin, input, and output (if you want to run this yourself, make sure you’ve installed protoc and followed the gRPC code [installation instructions](https://github.com/grpc/grpc/blob/v1.60.0/src/cpp/README.md#cmake) first):

​	为了简单起见，我们提供了一个 CMakeLists.txt，它为您运行 `protoc` ，并带有适当的插件、输入和输出（如果您想自己运行它，请确保您已安装 protoc 并首先按照 gRPC 代码安装说明进行操作）：

```sh
$ make route_guide.grpc.pb.o
```

which actually runs: 
​	实际上运行：

```sh
$ protoc -I ../../protos --grpc_out=. --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` ../../protos/route_guide.proto
$ protoc -I ../../protos --cpp_out=. ../../protos/route_guide.proto
```

Running this command generates the following files in your current directory:

​	运行此命令会在您的当前目录中生成以下文件：

- `route_guide.pb.h`, the header which declares your generated message classes
  `route_guide.pb.h` ，声明您生成的 message 类的头文件
- `route_guide.pb.cc`, which contains the implementation of your message classes
  `route_guide.pb.cc` ，其中包含您 message 类的实现
- `route_guide.grpc.pb.h`, the header which declares your generated service classes
  `route_guide.grpc.pb.h` ，声明您生成的 service 类的头文件
- `route_guide.grpc.pb.cc`, which contains the implementation of your service classes
  `route_guide.grpc.pb.cc` ，其中包含您 service 类的实现

These contain: 
​	它们包含：

- All the protocol buffer code to populate, serialize, and retrieve our request and response message types

  ​	填充、序列化和检索我们的请求和响应消息类型的所有协议缓冲区代码

- A class called `RouteGuide` that contains

  ​	一个名为 `RouteGuide` 的类，其中包含

  - a remote interface type (or *stub*) for clients to call with the methods defined in the `RouteGuide` service.
    供客户端调用的远程接口类型（或存根），其中包含在 `RouteGuide` 服务中定义的方法。
  - two abstract interfaces for servers to implement, also with the methods defined in the `RouteGuide` service.
    供服务器实现的两个抽象接口，其中也包含在 `RouteGuide` 服务中定义的方法。

### Creating the server 创建服务器

First let’s look at how we create a `RouteGuide` server. If you’re only interested in creating gRPC clients, you can skip this section and go straight to [Creating the client]({{< ref "/Languages/C/Basicstutorial#client" >}}) (though you might find it interesting anyway!).

​	首先，我们来看看如何创建一个 `RouteGuide` 服务器。如果您只对创建 gRPC 客户端感兴趣，可以跳过本部分，直接转到创建客户端（不过您可能还是会觉得它很有趣！）。

There are two parts to making our `RouteGuide` service do its job:

​	让我们的 `RouteGuide` 服务发挥作用有两个部分：

- Implementing the service interface generated from our service definition: doing the actual “work” of our service.
  实现从服务定义中生成的服务接口：执行服务的实际“工作”。
- Running a gRPC server to listen for requests from clients and return the service responses.
  运行 gRPC 服务器以侦听来自客户端的请求并返回服务响应。

You can find our example `RouteGuide` server in [examples/cpp/route_guide/route_guide_server.cc](https://github.com/grpc/grpc/blob/v1.60.0/examples/cpp/route_guide/route_guide_server.cc). Let’s take a closer look at how it works.

​	您可以在 examples/cpp/route_guide/route_guide_server.cc 中找到示例 `RouteGuide` 服务器。我们仔细看看它是如何工作的。

#### Implementing RouteGuide 实现 RouteGuide

As you can see, our server has a `RouteGuideImpl` class that implements the generated `RouteGuide::Service` interface:

​	如您所见，我们的服务器有一个 `RouteGuideImpl` 类，它实现了生成的 `RouteGuide::Service` 接口：

```cpp
class RouteGuideImpl final : public RouteGuide::Service {
...
}
```

In this case we’re implementing the *synchronous* version of `RouteGuide`, which provides our default gRPC server behaviour. It’s also possible to implement an asynchronous interface, `RouteGuide::AsyncService`, which allows you to further customize your server’s threading behaviour, though we won’t look at this in this tutorial.

​	在这种情况下，我们正在实现 `RouteGuide` 的同步版本，它提供了我们的默认 gRPC 服务器行为。还可以实现异步接口 `RouteGuide::AsyncService` ，它允许您进一步自定义服务器的线程行为，但我们不会在本教程中介绍这一点。

`RouteGuideImpl` implements all our service methods. Let’s look at the simplest type first, `GetFeature`, which just gets a `Point` from the client and returns the corresponding feature information from its database in a `Feature`.

​	 `RouteGuideImpl` 实现我们所有的服务方法。我们先来看最简单的类型 `GetFeature` ，它只是从客户端获取 `Point` ，并从其数据库中的 `Feature` 中返回相应的要素信息。

```cpp
Status GetFeature(ServerContext* context, const Point* point,
                  Feature* feature) override {
  feature->set_name(GetFeatureName(*point, feature_list_));
  feature->mutable_location()->CopyFrom(*point);
  return Status::OK;
}
```

The method is passed a context object for the RPC, the client’s `Point` protocol buffer request, and a `Feature` protocol buffer to fill in with the response information. In the method we populate the `Feature` with the appropriate information, and then `return` with an `OK` status to tell gRPC that we’ve finished dealing with the RPC and that the `Feature` can be returned to the client.

​	该方法传递一个 RPC 的上下文对象、客户端的 `Point` 协议缓冲区请求，以及一个 `Feature` 协议缓冲区，其中填充有响应信息。在此方法中，我们使用适当的信息填充 `Feature` ，然后使用 `OK` 状态填充 `return` ，以告知 gRPC 我们已完成处理 RPC，并且可以将 `Feature` 返回给客户端。

Note that all service methods can (and will!) be called from multiple threads at the same time. You have to make sure that your method implementations are thread safe. In our example, `feature_list_` is never changed after construction, so it is safe by design. But if `feature_list_` would change during the lifetime of the service, we would need to synchronize access to this member.

​	请注意，所有服务方法都可以（并且将会！）同时从多个线程调用。您必须确保您的方法实现是线程安全的。在我们的示例中， `feature_list_` 在构建后绝不会更改，因此设计上是安全的。但是，如果 `feature_list_` 在服务的生命周期内发生更改，我们需要同步对该成员的访问。

Now let’s look at something a bit more complicated - a streaming RPC. `ListFeatures` is a server-side streaming RPC, so we need to send back multiple `Feature`s to our client.

​	现在，我们来看一些更复杂的内容 - 流式 RPC。 `ListFeatures` 是服务器端流式 RPC，因此我们需要向我们的客户端发送多个 `Feature` 。

```cpp
Status ListFeatures(ServerContext* context, const Rectangle* rectangle,
                    ServerWriter<Feature>* writer) override {
  auto lo = rectangle->lo();
  auto hi = rectangle->hi();
  long left = std::min(lo.longitude(), hi.longitude());
  long right = std::max(lo.longitude(), hi.longitude());
  long top = std::max(lo.latitude(), hi.latitude());
  long bottom = std::min(lo.latitude(), hi.latitude());
  for (const Feature& f : feature_list_) {
    if (f.location().longitude() >= left &&
        f.location().longitude() <= right &&
        f.location().latitude() >= bottom &&
        f.location().latitude() <= top) {
      writer->Write(f);
    }
  }
  return Status::OK;
}
```

As you can see, instead of getting simple request and response objects in our method parameters, this time we get a request object (the `Rectangle` in which our client wants to find `Feature`s) and a special `ServerWriter` object. In the method, we populate as many `Feature` objects as we need to return, writing them to the `ServerWriter` using its `Write()` method. Finally, as in our simple RPC, we `return Status::OK` to tell gRPC that we’ve finished writing responses.

​	正如您所见，这次我们不是在方法参数中获取简单的请求和响应对象，而是获取一个请求对象（ `Rectangle` ，其中我们的客户端想要查找 `Feature` ）和一个特殊的 `ServerWriter` 对象。在此方法中，我们填充尽可能多的 `Feature` 对象以供返回，并使用其 `Write()` 方法将它们写入 `ServerWriter` 。最后，与在我们的简单 RPC 中一样，我们 `return Status::OK` 以告诉 gRPC 我们已完成写入响应。

If you look at the client-side streaming method `RecordRoute` you’ll see it’s quite similar, except this time we get a `ServerReader` instead of a request object and a single response. We use the `ServerReader`s `Read()` method to repeatedly read in our client’s requests to a request object (in this case a `Point`) until there are no more messages: the server needs to check the return value of `Read()` after each call. If `true`, the stream is still good and it can continue reading; if `false` the message stream has ended.

​	如果您查看客户端流方法 `RecordRoute` ，您会发现它非常相似，只是这次我们获取的是 `ServerReader` 而不是请求对象和单个响应。我们使用 `ServerReader` 的 `Read()` 方法反复读取客户端的请求到请求对象（在本例中为 `Point` ），直到没有更多消息：服务器需要在每次调用后检查 `Read()` 的返回值。如果 `true` ，则流仍然有效，并且可以继续读取；如果 `false` ，则消息流已结束。

```cpp
while (stream->Read(&point)) {
  ...//process client input
}
```

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`.

​	最后，我们来看一下我们的双向流 RPC `RouteChat()` 。

```cpp
Status RouteChat(ServerContext* context,
                  ServerReaderWriter<RouteNote, RouteNote>* stream) override {
  RouteNote note;
  while (stream->Read(&note)) {
    std::unique_lock<std::mutex> lock(mu_);
    for (const RouteNote& n : received_notes_) {
      if (n.location().latitude() == note.location().latitude() &&
          n.location().longitude() == note.location().longitude()) {
        stream->Write(n);
      }
    }
    received_notes_.push_back(note);
  }

  return Status::OK;
}
```

This time we get a `ServerReaderWriter` that can be used to read *and* write messages. The syntax for reading and writing here is exactly the same as for our client-streaming and server-streaming methods. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	这次我们得到一个 `ServerReaderWriter` ，可用于读写消息。此处用于读写的语法与我们的客户端流和服务器流方法完全相同。尽管每一方始终会按写入顺序收到另一方的消息，但客户端和服务器都可以按任何顺序读写——流完全独立地运行。

Note that since `received_notes_` is an instance variable and can be accessed by multiple threads, we use a mutex lock here to guarantee exclusive access.

​	请注意，由于 `received_notes_` 是一个实例变量，多个线程可以访问它，因此我们在此处使用互斥锁来保证独占访问。

#### Starting the server 启动服务器

Once we’ve implemented all our methods, we also need to start up a gRPC server so that clients can actually use our service. The following snippet shows how we do this for our `RouteGuide` service:

​	实现所有方法后，我们还需要启动一个 gRPC 服务器，以便客户端实际使用我们的服务。以下代码段展示了我们如何为 `RouteGuide` 服务执行此操作：

```cpp
void RunServer(const std::string& db_path) {
  std::string server_address("0.0.0.0:50051");
  RouteGuideImpl service(db_path);

  ServerBuilder builder;
  builder.AddListeningPort(server_address, grpc::InsecureServerCredentials());
  builder.RegisterService(&service);
  std::unique_ptr<Server> server(builder.BuildAndStart());
  std::cout << "Server listening on " << server_address << std::endl;
  server->Wait();
}
```

As you can see, we build and start our server using a `ServerBuilder`. To do this, we:

​	如您所见，我们使用 `ServerBuilder` 构建并启动服务器。为此，我们：

1. Create an instance of our service implementation class `RouteGuideImpl`.
   创建一个服务实现类 `RouteGuideImpl` 的实例。
2. Create an instance of the factory `ServerBuilder` class.
   创建工厂 `ServerBuilder` 类的实例。
3. Specify the address and port we want to use to listen for client requests using the builder’s `AddListeningPort()` method.
   使用构建器的 `AddListeningPort()` 方法指定我们想要用于侦听客户端请求的地址和端口。
4. Register our service implementation with the builder.
   使用构建器注册我们的服务实现。
5. Call `BuildAndStart()` on the builder to create and start an RPC server for our service.
   在构建器上调用 `BuildAndStart()` 以创建和启动我们服务的 RPC 服务器。
6. Call `Wait()` on the server to do a blocking wait until process is killed or `Shutdown()` is called.
   在服务器上调用 `Wait()` 以执行阻塞等待，直到进程被终止或调用 `Shutdown()` 。

### Creating the client 创建客户端

In this section, we’ll look at creating a C++ client for our `RouteGuide` service. You can see our complete example client code in [examples/cpp/route_guide/route_guide_client.cc](https://github.com/grpc/grpc/blob/v1.60.0/examples/cpp/route_guide/route_guide_client.cc).

​	在本节中，我们将研究为我们的 `RouteGuide` 服务创建 C++ 客户端。您可以在 examples/cpp/route_guide/route_guide_client.cc 中看到我们完整的示例客户端代码。

#### Creating a stub 创建存根

To call service methods, we first need to create a *stub*.

​	要调用服务方法，我们首先需要创建一个存根。

First we need to create a gRPC *channel* for our stub, specifying the server address and port we want to connect to - in our case we’ll use no SSL:

​	首先，我们需要为我们的存根创建一个 gRPC 通道，指定我们要连接的服务器地址和端口 - 在我们的案例中，我们将不使用 SSL：

```cpp
grpc::CreateChannel("localhost:50051", grpc::InsecureChannelCredentials());
```

#### Note 注意

In order to set additional options for the *channel*, use the `grpc::CreateCustomChannel()` api with any special channel arguments - `grpc::ChannelArguments`.
为了为通道设置其他选项，请使用 `grpc::CreateCustomChannel()` api 和任何特殊通道参数 - `grpc::ChannelArguments` 。

Now we can use the channel to create our stub using the `NewStub` method provided in the `RouteGuide` class we generated from our `.proto`.

​	现在，我们可以使用该通道来创建存根，方法是使用我们从 `.proto` 生成的 `RouteGuide` 类中提供的 `NewStub` 方法。

```cpp
public:
 RouteGuideClient(std::shared_ptr<ChannelInterface> channel,
                  const std::string& db)
     : stub_(RouteGuide::NewStub(channel)) {
   ...
 }
```

#### Calling service methods 调用服务方法

Now let’s look at how we call our service methods. Note that in this tutorial we’re calling the *blocking/synchronous* versions of each method: this means that the RPC call waits for the server to respond, and will either return a response or raise an exception.

​	现在，我们来看看如何调用我们的服务方法。请注意，在本教程中，我们调用的是每个方法的阻塞/同步版本：这意味着 RPC 调用会等待服务器响应，并且会返回响应或引发异常。

##### Simple RPC 简单 RPC

Calling the simple RPC `GetFeature` is nearly as straightforward as calling a local method.

​	调用简单 RPC `GetFeature` 几乎与调用本地方法一样简单。

```cpp
Point point;
Feature feature;
point = MakePoint(409146138, -746188906);
GetOneFeature(point, &feature);

...

bool GetOneFeature(const Point& point, Feature* feature) {
  ClientContext context;
  Status status = stub_->GetFeature(&context, point, feature);
  ...
}
```

As you can see, we create and populate a request protocol buffer object (in our case `Point`), and create a response protocol buffer object for the server to fill in. We also create a `ClientContext` object for our call - you can optionally set RPC configuration values on this object, such as deadlines, though for now we’ll use the default settings. Note that you cannot reuse this object between calls. Finally, we call the method on the stub, passing it the context, request, and response. If the method returns `OK`, then we can read the response information from the server from our response object.

​	如您所见，我们创建并填充请求协议缓冲区对象（在本例中为 `Point` ），并为服务器创建响应协议缓冲区对象以供其填写。我们还为调用创建 `ClientContext` 对象 - 您可以在此对象上设置 RPC 配置值，例如截止时间，但现在我们将使用默认设置。请注意，您不能在调用之间重复使用此对象。最后，我们在存根上调用该方法，将上下文、请求和响应传递给它。如果该方法返回 `OK` ，那么我们就可以从响应对象中读取服务器的响应信息。

```cpp
std::cout << "Found feature called " << feature->name()  << " at "
          << feature->location().latitude()/kCoordFactor_ << ", "
          << feature->location().longitude()/kCoordFactor_ << std::endl;
```

##### Streaming RPCs 流式 RPC

Now let’s look at our streaming methods. If you’ve already read [Creating the server]({{< ref "/Languages/C/Basicstutorial#server" >}}) some of this may look very familiar - streaming RPCs are implemented in a similar way on both sides. Here’s where we call the server-side streaming method `ListFeatures`, which returns a stream of geographical `Feature`s:

​	现在让我们看看我们的流式方法。如果您已经阅读了创建服务器，其中一些可能看起来非常熟悉 - 流式 RPC 在双方以类似的方式实现。我们在这里调用服务器端流式方法 `ListFeatures` ，它返回一个地理 `Feature` 流：

```cpp
std::unique_ptr<ClientReader<Feature> > reader(
    stub_->ListFeatures(&context, rect));
while (reader->Read(&feature)) {
  std::cout << "Found feature called "
            << feature.name() << " at "
            << feature.location().latitude()/kCoordFactor_ << ", "
            << feature.location().longitude()/kCoordFactor_ << std::endl;
}
Status status = reader->Finish();
```

Instead of passing the method a context, request, and response, we pass it a context and request and get a `ClientReader` object back. The client can use the `ClientReader` to read the server’s responses. We use the `ClientReader`s `Read()` method to repeatedly read in the server’s responses to a response protocol buffer object (in this case a `Feature`) until there are no more messages: the client needs to check the return value of `Read()` after each call. If `true`, the stream is still good and it can continue reading; if `false` the message stream has ended. Finally, we call `Finish()` on the stream to complete the call and get our RPC status.

​	我们不是向该方法传递上下文、请求和响应，而是向它传递上下文和请求，并返回一个 `ClientReader` 对象。客户端可以使用 `ClientReader` 读取服务器的响应。我们使用 `ClientReader` s `Read()` 方法重复读取服务器对响应协议缓冲区对象（在本例中为 `Feature` ）的响应，直到没有更多消息：客户端需要在每次调用后检查 `Read()` 的返回值。如果 `true` ，流仍然有效，并且可以继续读取；如果 `false` ，消息流已结束。最后，我们对流调用 `Finish()` 以完成调用并获取我们的 RPC 状态。

The client-side streaming method `RecordRoute` is similar, except there we pass the method a context and response object and get back a `ClientWriter`.

​	客户端流式方法 `RecordRoute` 类似，不同之处在于我们向该方法传递上下文和响应对象，并返回一个 `ClientWriter` 。

```cpp
std::unique_ptr<ClientWriter<Point> > writer(
    stub_->RecordRoute(&context, &stats));
for (int i = 0; i < kPoints; i++) {
  const Feature& f = feature_list_[feature_distribution(generator)];
  std::cout << "Visiting point "
            << f.location().latitude()/kCoordFactor_ << ", "
            << f.location().longitude()/kCoordFactor_ << std::endl;
  if (!writer->Write(f.location())) {
    // Broken stream.
    break;
  }
  std::this_thread::sleep_for(std::chrono::milliseconds(
      delay_distribution(generator)));
}
writer->WritesDone();
Status status = writer->Finish();
if (status.IsOk()) {
  std::cout << "Finished trip with " << stats.point_count() << " points\n"
            << "Passed " << stats.feature_count() << " features\n"
            << "Travelled " << stats.distance() << " meters\n"
            << "It took " << stats.elapsed_time() << " seconds"
            << std::endl;
} else {
  std::cout << "RecordRoute rpc failed." << std::endl;
}
```

Once we’ve finished writing our client’s requests to the stream using `Write()`, we need to call `WritesDone()` on the stream to let gRPC know that we’ve finished writing, then `Finish()` to complete the call and get our RPC status. If the status is `OK`, our response object that we initially passed to `RecordRoute()` will be populated with the server’s response.

​	在使用 `Write()` 将客户端请求写入流后，我们需要在流上调用 `WritesDone()` 以便让 gRPC 知道我们已完成写入，然后调用 `Finish()` 以完成调用并获取我们的 RPC 状态。如果状态为 `OK` ，我们最初传递给 `RecordRoute()` 的响应对象将填充服务器的响应。

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`. In this case, we just pass a context to the method and get back a `ClientReaderWriter`, which we can use to both write and read messages.

​	最后，我们来看一下双向流式 RPC `RouteChat()` 。在这种情况下，我们只需将上下文传递给该方法，然后返回一个 `ClientReaderWriter` ，我们可以使用它来写入和读取消息。

```cpp
std::shared_ptr<ClientReaderWriter<RouteNote, RouteNote> > stream(
    stub_->RouteChat(&context));
```

The syntax for reading and writing here is exactly the same as for our client-streaming and server-streaming methods. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	此处用于读取和写入的语法与我们的客户端流式传输和服务器流式传输方法完全相同。尽管每一方总是会按写入顺序收到另一方的消息，但客户端和服务器都可以按任何顺序读取和写入——流完全独立地运行。

### Try it out! 试一试！

Build the client and server:

​	构建客户端和服务器：

```sh
$ make
```

Run the server: 
​	运行服务器：

```sh
$ ./route_guide_server --db_path=path/to/route_guide_db.json
```

From a different terminal, run the client:

​	从另一个终端运行客户端：

```sh
$ ./route_guide_client --db_path=path/to/route_guide_db.json
```
