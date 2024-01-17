+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/php/basics/](https://grpc.io/docs/languages/php/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC in PHP.

​	gRPC 在 PHP 中的基本教程介绍。



This tutorial provides a basic PHP programmer’s introduction to working with gRPC.

​	本教程为 PHP 程序员提供了使用 gRPC 的基本入门知识。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a .proto file.
  在 .proto 文件中定义服务。
- Generate client code using the protocol buffer compiler.
  使用协议缓冲区编译器生成客户端代码。
- Use the PHP gRPC API to write a simple client for your service.
  使用 PHP gRPC API 为您的服务编写一个简单的客户端。

It assumes a passing familiarity with [protocol buffers](https://protobuf.dev/overview). Note that the example in this tutorial uses the proto2 version of the protocol buffers language.

​	它假定您对协议缓冲区有一定了解。请注意，本教程中的示例使用协议缓冲区语言的 proto2 版本。

Also note that currently, you can only create clients in PHP for gRPC services. Use [another language](https://grpc.io/docs/languages/) to create a gRPC server.

​	另请注意，目前您只能在 PHP 中为 gRPC 服务创建客户端。使用另一种语言创建 gRPC 服务器。

### Why use gRPC? 为什么要使用 gRPC？

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	使用 gRPC，我们可以在 `.proto` 文件中定义一次服务，并在 gRPC 支持的任何语言中生成客户端和服务器，然后可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行这些客户端和服务器——gRPC 会为您处理不同语言和环境之间的所有通信复杂性。我们还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。

### Example code and setup 示例代码和设置

The example code for our tutorial is in [grpc/grpc/examples/php/route_guide](https://github.com/grpc/grpc/tree/v1.60.0/examples/php/route_guide). To download the example, clone the `grpc` repository and its submodules by running the following command:

​	本教程的示例代码位于 grpc/grpc/examples/php/route_guide 中。要下载示例，请克隆 `grpc` 存储库及其子模块，方法是运行以下命令：

```sh
$ git clone --recurse-submodules -b v1.60.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc
```

You need the `grpc-php-plugin` to help you compile `.proto` files. Build it from source as follows:

​	您需要 `grpc-php-plugin` 来帮助您编译 `.proto` 文件。从源代码构建它，如下所示：

```sh
$ cd grpc
$ mkdir -p cmake/build
$ pushd cmake/build
$ cmake ../..
$ make protoc grpc_php_plugin
$ popd
```

Then change to route guide directory and compile the example’s `.proto` files:

​	然后切换到路线指南目录并编译示例的 `.proto` 文件：

```sh
$ cd examples/php/route_guide
$ ./route_guide_proto_gen.sh
```

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

You also should have the relevant tools installed to generate the client interface code (and a server in another language, for testing). You can obtain the latter by following [these setup instructions]({{< ref "/Languages/Node/Basicstutorial" >}}), for example.

​	您还应该安装相关工具来生成客户端接口代码（以及另一种语言中的服务器，用于测试）。例如，您可以按照这些设置说明获取后者。

### Try it out! 试一试！

To try the sample app, we need a gRPC server running locally. Let’s compile and run, for example, the Node.js server in this repository:

​	要试用示例应用，我们需要在本地运行一个 gRPC 服务器。让我们编译并运行此存储库中的 Node.js 服务器，例如：

```sh
$ cd ../../node
$ npm install
$ cd dynamic_codegen/route_guide
$ nodejs ./route_guide_server.js --db_path=route_guide_db.json
```

Run the PHP client (in a different terminal):

​	运行 PHP 客户端（在另一个终端中）：

```sh
$ ./run_route_guide_client.sh
```

The next sections guide you step-by-step through how this proto service is defined, how to generate a client library from it, and how to create a client stub that uses that library.

​	下面的部分将逐步指导您完成如何定义此 proto 服务、如何从中生成客户端库以及如何创建使用该库的客户端存根。

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

### Generating client code 生成客户端代码

The PHP client stub implementation of the proto files can be generated by the gRPC PHP Protoc Plugin. To compile the plugin:

​	proto 文件的 PHP 客户端存根实现可以通过 gRPC PHP Protoc 插件生成。要编译插件：

```sh
$ make grpc_php_plugin
```

To generate the client stub implementation .php file:

​	要生成客户端存根实现 .php 文件：

```sh
$ cd grpc
$ protoc --proto_path=examples/protos \
  --php_out=examples/php/route_guide \
  --grpc_out=examples/php/route_guide \
  --plugin=protoc-gen-grpc=bins/opt/grpc_php_plugin \
  ./examples/protos/route_guide.proto
```

or running the helper script under the `grpc/example/php/route_guide` directory if you build grpc-php-plugin by source:

​	或在 `grpc/example/php/route_guide` 目录下运行帮助程序脚本（如果您通过源代码构建 grpc-php-plugin）：

```sh
$ ./route_guide_proto_gen.sh
```

A number of files will be generated in the `examples/php/route_guide` directory. You do not need to modify those files.

​	将在 `examples/php/route_guide` 目录中生成许多文件。您无需修改这些文件。

To load these generated files, add this section to your `composer.json` file under `examples/php` directory

​	要加载这些生成的文件，请将此部分添加到 `examples/php` 目录下的 `composer.json` 文件中

```json
  "autoload": {
    "psr-4": {
      "": "route_guide/"
    }
  }
```

The file contains: 
​	该文件包含：

- All the protocol buffer code to populate, serialize, and retrieve our request and response message types.
  用于填充、序列化和检索请求和响应消息类型的所有协议缓冲区代码。
- A class called `Routeguide\RouteGuideClient` that lets clients call the methods defined in the `RouteGuide` service.
  一个名为 `Routeguide\RouteGuideClient` 的类，允许客户端调用 `RouteGuide` 服务中定义的方法。

### Creating the client 创建客户端

In this section, we’ll look at creating a PHP client for our `RouteGuide` service. You can see our complete example client code in [examples/php/route_guide/route_guide_client.php](https://github.com/grpc/grpc/blob/v1.60.0/examples/php/route_guide/route_guide_client.php).

​	在本部分，我们将着眼于为我们的 `RouteGuide` 服务创建一个 PHP 客户端。您可以在 examples/php/route_guide/route_guide_client.php 中看到我们完整的示例客户端代码。

#### Constructing a client object 构造客户端对象

To call service methods, we first need to create a client object, an instance of the generated `RouteGuideClient` class. The constructor of the class expects the server address and port we want to connect to:

​	要调用服务方法，我们首先需要创建一个客户端对象，即生成的 `RouteGuideClient` 类的实例。该类的构造函数需要我们想要连接到的服务器地址和端口：

```php
$client = new Routeguide\RouteGuideClient('localhost:50051', [
    'credentials' => Grpc\ChannelCredentials::createInsecure(),
]);
```

#### Calling service methods 调用服务方法

Now let’s look at how we call our service methods.

​	现在，我们来看看如何调用服务方法。

##### Simple RPC 简单 RPC

Calling the simple RPC `GetFeature` is nearly as straightforward as calling a local asynchronous method.

​	调用简单的 RPC `GetFeature` 与调用本地异步方法几乎一样简单。

```php
$point = new Routeguide\Point();
$point->setLatitude(409146138);
$point->setLongitude(-746188906);
list($feature, $status) = $client->GetFeature($point)->wait();
```

As you can see, we create and populate a request object, i.e. an `Routeguide\Point` object. Then, we call the method on the stub, passing it the request object. If there is no error, then we can read the response information from the server from our response object, i.e. an `Routeguide\Feature` object.

​	如您所见，我们创建一个请求对象并填充它，即一个 `Routeguide\Point` 对象。然后，我们调用存根上的方法，将请求对象传递给它。如果没有错误，那么我们就可以从响应对象中读取服务器的响应信息，即一个 `Routeguide\Feature` 对象。

```php
print sprintf("Found %s \n  at %f, %f\n", $feature->getName(),
              $feature->getLocation()->getLatitude() / COORD_FACTOR,
              $feature->getLocation()->getLongitude() / COORD_FACTOR);
```

##### Streaming RPCs 流式 RPC

Now let’s look at our streaming methods. Here’s where we call the server-side streaming method `ListFeatures`, which returns a stream of geographical `Feature`s:

​	现在让我们看看我们的流方法。我们在这里调用服务器端流方法 `ListFeatures` ，它返回一个地理 `Feature` 流：

```php
$lo_point = new Routeguide\Point();
$hi_point = new Routeguide\Point();

$lo_point->setLatitude(400000000);
$lo_point->setLongitude(-750000000);
$hi_point->setLatitude(420000000);
$hi_point->setLongitude(-730000000);

$rectangle = new Routeguide\Rectangle();
$rectangle->setLo($lo_point);
$rectangle->setHi($hi_point);

$call = $client->ListFeatures($rectangle);
// an iterator over the server streaming responses
$features = $call->responses();
foreach ($features as $feature) {
  // process each feature
} // the loop will end when the server indicates there is no more responses to be sent.
```

The `$call->responses()` method call returns an iterator. When the server sends a response, a `$feature` object will be returned in the `foreach` loop, until the server indiciates that there will be no more responses to be sent.

​	方法调用 `$call->responses()` 返回一个迭代器。当服务器发送响应时， `$feature` 对象将在 `foreach` 循环中返回，直到服务器指示不再发送任何响应。

The client-side streaming method `RecordRoute` is similar, except that we call `$call->write($point)` for each point we want to write from the client side and get back a `Routeguide\RouteSummary`.

​	客户端流方法 `RecordRoute` 类似，不同之处在于我们为要从客户端写入的每个点调用 `$call->write($point)` ，并返回 `Routeguide\RouteSummary` 。

```php
$call = $client->RecordRoute();

for ($i = 0; $i < $num_points; $i++) {
  $point = new Routeguide\Point();
  $point->setLatitude($lat);
  $point->setLongitude($long);
  $call->write($point);
}

list($route_summary, $status) = $call->wait();
```

Finally, let’s look at our bidirectional streaming RPC `routeChat()`. In this case, we just pass a context to the method and get back a `BidiStreamingCall` stream object, which we can use to both write and read messages.

​	最后，我们来看一下双向流式 RPC `routeChat()` 。在这种情况下，我们只需将上下文传递给该方法，然后返回 `BidiStreamingCall` 流对象，我们可以使用该对象来写入和读取消息。

```php
$call = $client->RouteChat();
```

To write messages from the client:

​	要从客户端写入消息：

```php
foreach ($notes as $n) {
  $point = new Routeguide\Point();
  $point->setLatitude($lat = $n[0]);
  $point->setLongitude($long = $n[1]);

  $route_note = new Routeguide\RouteNote();
  $route_note->setLocation($point);
  $route_note->setMessage($message = $n[2]);
  $call->write($route_note);
}
$call->writesDone();
```

To read messages from the server:

​	要从服务器读取消息：

```php
while ($route_note_reply = $call->read()) {
  // process $route_note_reply
}
```

Each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	每一方总是按顺序收到另一方的消息，客户端和服务器都可以按任何顺序读取和写入——流完全独立地运行。
