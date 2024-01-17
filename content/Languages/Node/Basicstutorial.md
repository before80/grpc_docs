+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/node/basics/](https://grpc.io/docs/languages/node/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC in Node.

​	Node 中 gRPC 的基本教程介绍。



This tutorial provides a basic Node.js programmer’s introduction to working with gRPC.

​	本教程为 Node.js 程序员提供了使用 gRPC 的基本介绍。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a `.proto` file.
  在 `.proto` 文件中定义服务。
- Use the Node.js gRPC API to write a simple client and server for your service.
  使用 Node.js gRPC API 为您的服务编写简单的客户端和服务器。

It assumes that you have read the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}) and are familiar with [protocol buffers](https://protobuf.dev/overview). Note that the example in this tutorial uses the [proto3](https://github.com/google/protobuf/releases) version of the protocol buffers language. You can find out more in the [proto3 language guide](https://protobuf.dev/programming-guides/proto3).

​	它假定您已阅读 gRPC 简介并且熟悉协议缓冲区。请注意，本教程中的示例使用协议缓冲区语言的 proto3 版本。您可以在 proto3 语言指南中了解更多信息。

### Why use gRPC? 为什么要使用 gRPC？

Our example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

​	我们的示例是一个简单的路线映射应用程序，它允许客户端获取有关其路线上的功能的信息、创建其路线的摘要以及与服务器和其他客户端交换路线信息，例如交通更新。

With gRPC we can define our service once in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet — all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating.

​	使用 gRPC，我们可以在 `.proto` 文件中定义一次服务，并在 gRPC 支持的任何语言中生成客户端和服务器，然后可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行这些客户端和服务器——gRPC 会为您处理不同语言和环境之间的所有通信复杂性。我们还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。

### Example code and setup 示例代码和设置

The example code for our tutorial is in [grpc/grpc-node/examples/routeguide/dynamic_codegen](https://github.com/grpc/grpc-node/tree/@grpc/grpc-js@1.9.0/examples/routeguide/dynamic_codegen). As you’ll see if you look at the repository, there’s also a very similar-looking example in [grpc/grpc-node/examples/routeguide/static_codegen](https://github.com/grpc/grpc-node/tree/@grpc/grpc-js@1.9.0/examples/routeguide/static_codegen). We have two versions of our route guide example because there are two ways to generate the code needed to work with protocol buffers in Node.js - one approach uses `Protobuf.js` to dynamically generate the code at runtime, the other uses code statically generated using the protocol buffer compiler `protoc`. The examples behave identically, and either server can be used with either client. As suggested by the directory name, we’ll be using the version with dynamically generated code in this document, but feel free to look at the static code example too.

​	我们教程的示例代码位于 grpc/grpc-node/examples/routeguide/dynamic_codegen。如果您查看存储库，您会发现 grpc/grpc-node/examples/routeguide/static_codegen 中还有一个非常相似的示例。我们有两个版本的路线指南示例，因为有两种方法可以在 Node.js 中生成与协议缓冲区配合使用所需的代码 - 一种方法使用 `Protobuf.js` 在运行时动态生成代码，另一种方法使用协议缓冲区编译器 `protoc` 静态生成的代码。这些示例的行为完全相同，任何服务器都可以与任何客户端配合使用。顾名思义，我们将在本文档中使用动态生成代码的版本，但您也可以随意查看静态代码示例。

To download the example, clone the `grpc` repository by running the following command:

​	要下载示例，请通过运行以下命令克隆 `grpc` 存储库：

```sh
$ git clone -b @grpc/grpc-js@1.9.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc-node
$ cd grpc
```

Then change your current directory to `examples`:

​	然后将当前目录更改为 `examples` ：

```sh
$ cd examples
```

You also should have the relevant tools installed to generate the server and client interface code - if you don’t already, follow the setup instructions in [Quick start]({{< ref "/Languages/Node/Quickstart" >}}).

​	您还应该安装相关工具来生成服务器和客户端接口代码 - 如果您还没有安装，请按照快速入门中的设置说明进行操作。

### Defining the service 定义服务

Our first step (as you’ll know from the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}})) is to define the gRPC *service* and the method *request* and *response* types using [protocol buffers](https://protobuf.dev/overview). You can see the complete .proto file in [`examples/protos/route_guide.proto`](https://github.com/grpc/grpc-node//blob/@grpc/grpc-js@1.9.0/examples/protos/route_guide.proto).

​	我们的第一步（如您从 gRPC 简介中了解到的）是使用协议缓冲区定义 gRPC 服务以及方法请求和响应类型。您可以在 `examples/protos/route_guide.proto` 中看到完整的 .proto 文件。

To define a service, you specify a named `service` in your `.proto` file:

​	要定义服务，您需要在 `.proto` 文件中指定一个命名的 `service` ：

```protobuf
service RouteGuide {
   ...
}
```

Then you define `rpc` methods inside your service definition, specifying their request and response types. gRPC lets you define four kinds of service methods, all of which are used in the `RouteGuide` service:

​	然后，您可以在服务定义中定义 `rpc` 方法，并指定其请求和响应类型。gRPC 允许您定义四种服务方法，所有这些方法都用于 `RouteGuide` 服务：

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

### Loading service descriptors from proto files 从 proto 文件加载服务描述符

The Node.js library dynamically generates service descriptors and client stub definitions from `.proto` files loaded at runtime.

​	Node.js 库会动态生成服务描述符和客户端存根定义，这些定义来自运行时加载的 `.proto` 文件。

To load a `.proto` file, simply `require` the gRPC proto loader library and use its `loadSync()` method, then pass the output to the gRPC library’s `loadPackageDefinition` method:

​	要加载 `.proto` 文件，只需 `require` gRPC proto 加载器库并使用其 `loadSync()` 方法，然后将输出传递给 gRPC 库的 `loadPackageDefinition` 方法：

```js
var PROTO_PATH = __dirname + '/../../protos/route_guide.proto';
var grpc = require('@grpc/grpc-js');
var protoLoader = require('@grpc/proto-loader');
// Suggested options for similarity to existing grpc.load behavior
var packageDefinition = protoLoader.loadSync(
    PROTO_PATH,
    {keepCase: true,
     longs: String,
     enums: String,
     defaults: true,
     oneofs: true
    });
var protoDescriptor = grpc.loadPackageDefinition(packageDefinition);
// The protoDescriptor object has the full package hierarchy
var routeguide = protoDescriptor.routeguide;
```

Once you’ve done this, the stub constructor is in the `routeguide` namespace (`protoDescriptor.routeguide.RouteGuide`) and the service descriptor (which is used to create a server) is a property of the stub (`protoDescriptor.routeguide.RouteGuide.service`);

​	完成此操作后，存根构造函数位于 `routeguide` 命名空间中 ( `protoDescriptor.routeguide.RouteGuide` )，服务描述符（用于创建服务器）是存根的属性 ( `protoDescriptor.routeguide.RouteGuide.service` )；

### Creating the server 创建服务器

First let’s look at how we create a `RouteGuide` server. If you’re only interested in creating gRPC clients, you can skip this section and go straight to [Creating the client]({{< ref "/Languages/Node/Basicstutorial#client" >}}) (though you might find it interesting anyway!).

​	首先，我们来看看如何创建一个 `RouteGuide` 服务器。如果您只对创建 gRPC 客户端感兴趣，可以跳过本部分，直接转到创建客户端（不过您可能还是会觉得它很有趣！）。

There are two parts to making our `RouteGuide` service do its job:

​	让我们的 `RouteGuide` 服务发挥作用有两个部分：

- Implementing the service interface generated from our service definition: doing the actual “work” of our service.
  实现从服务定义中生成的服务接口：执行服务的实际“工作”。
- Running a gRPC server to listen for requests from clients and return the service responses.
  运行 gRPC 服务器以侦听来自客户端的请求并返回服务响应。

You can find our example `RouteGuide` server in [examples/routeguide/dynamic_codegen/route_guide_server.js](https://github.com/grpc/grpc-node/blob/@grpc/grpc-js@1.9.0/examples/routeguide/dynamic_codegen/route_guide_server.js). Let’s take a closer look at how it works.

​	您可以在 examples/routeguide/dynamic_codegen/route_guide_server.js 中找到我们的示例 `RouteGuide` 服务器。我们仔细看看它是如何工作的。

#### Implementing RouteGuide 实现 RouteGuide

As you can see, our server has a `Server` constructor generated from the `RouteGuide.service` descriptor object

​	如您所见，我们的服务器有一个 `Server` 构造函数，该构造函数由 `RouteGuide.service` 描述符对象生成

```js
var Server = new grpc.Server();
```

In this case we’re implementing the *asynchronous* version of `RouteGuide`, which provides our default gRPC server behavior.

​	在这种情况下，我们正在实现 `RouteGuide` 的异步版本，该版本提供了我们的默认 gRPC 服务器行为。

The functions in `route_guide_server.js` implement all our service methods. Let’s look at the simplest type first, `getFeature`, which just gets a `Point` from the client and returns the corresponding feature information from its database in a `Feature`.

​	 `route_guide_server.js` 中的函数实现了我们所有的服务方法。我们先来看最简单的类型 `getFeature` ，它只是从客户端获取 `Point` ，并从其数据库中的 `Feature` 中返回相应的要素信息。

```js
function checkFeature(point) {
  var feature;
  // Check if there is already a feature object for the given point
  for (var i = 0; i < feature_list.length; i++) {
    feature = feature_list[i];
    if (feature.location.latitude === point.latitude &&
        feature.location.longitude === point.longitude) {
      return feature;
    }
  }
  var name = '';
  feature = {
    name: name,
    location: point
  };
  return feature;
}
function getFeature(call, callback) {
  callback(null, checkFeature(call.request));
}
```

The method is passed a call object for the RPC, which has the `Point` parameter as a property, and a callback to which we can pass our returned `Feature`. In the method body we populate a `Feature` corresponding to the given point and pass it to the callback, with a null first parameter to indicate that there is no error.

​	该方法传递一个用于 RPC 的调用对象，该对象具有 `Point` 参数作为属性，以及一个回调，我们可以将返回的 `Feature` 传递给它。在方法体中，我们填充一个与给定点对应的 `Feature` ，并将其传递给回调，第一个参数为 null，表示没有错误。

Now let’s look at something a bit more complicated - a streaming RPC. `listFeatures` is a server-side streaming RPC, so we need to send back multiple `Feature`s to our client.

​	现在，我们来看一些更复杂的内容 - 流式 RPC。 `listFeatures` 是服务器端流式 RPC，因此我们需要向我们的客户端发送多个 `Feature` 。

```js
function listFeatures(call) {
  var lo = call.request.lo;
  var hi = call.request.hi;
  var left = _.min([lo.longitude, hi.longitude]);
  var right = _.max([lo.longitude, hi.longitude]);
  var top = _.max([lo.latitude, hi.latitude]);
  var bottom = _.min([lo.latitude, hi.latitude]);
  // For each feature, check if it is in the given bounding box
  _.each(feature_list, function(feature) {
    if (feature.name === '') {
      return;
    }
    if (feature.location.longitude >= left &&
        feature.location.longitude <= right &&
        feature.location.latitude >= bottom &&
        feature.location.latitude <= top) {
      call.write(feature);
    }
  });
  call.end();
}
```

As you can see, instead of getting the call object and callback in our method parameters, this time we get a `call` object that implements the `Writable` interface. In the method, we create as many `Feature` objects as we need to return, writing them to the `call` using its `write()` method. Finally, we call `call.end()` to indicate that we have sent all messages.

​	如您所见，这次我们不是在方法参数中获取调用对象和回调，而是获取一个实现了 `Writable` 接口的 `call` 对象。在该方法中，我们创建任意数量需要返回的 `Feature` 对象，并使用其 `write()` 方法将它们写入 `call` 。最后，我们调用 `call.end()` 以指示我们已发送所有消息。

If you look at the client-side streaming method `RecordRoute` you’ll see it’s quite similar to the unary call, except this time the `call` parameter implements the `Reader` interface. The `call`’s `'data'` event fires every time there is new data, and the `'end'` event fires when all data has been read. Like the unary case, we respond by calling the callback

​	如果您查看客户端流方法 `RecordRoute` ，您会发现它与一元调用非常相似，只是这次 `call` 参数实现了 `Reader` 接口。每当有新数据时， `call` 的 `'data'` 事件都会触发，并且在读取所有数据时， `'end'` 事件会触发。与一元情况一样，我们通过调用回调来响应

```js
call.on('data', function(point) {
  // Process user data
});
call.on('end', function() {
  callback(null, result);
});
```

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`.

​	最后，我们来看一下我们的双向流 RPC `RouteChat()` 。

```js
function routeChat(call) {
  call.on('data', function(note) {
    var key = pointKey(note.location);
    /* For each note sent, respond with all previous notes that correspond to
     * the same point */
    if (route_notes.hasOwnProperty(key)) {
      _.each(route_notes[key], function(note) {
        call.write(note);
      });
    } else {
      route_notes[key] = [];
    }
    // Then add the new note to the list
    route_notes[key].push(JSON.parse(JSON.stringify(note)));
  });
  call.on('end', function() {
    call.end();
  });
}
```

This time we get a `call` implementing `Duplex` that can be used to read *and* write messages. The syntax for reading and writing here is exactly the same as for our client-streaming and server-streaming methods. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	这次我们得到一个实现 `call` 的 `Duplex` ，它可用于读写消息。此处读写语法与我们的客户端流和服务器流方法完全相同。尽管每一方总是按写入顺序接收另一方的消息，但客户端和服务器都可以按任何顺序读写——流完全独立地运行。

#### Starting the server 启动服务器

Once we’ve implemented all our methods, we also need to start up a gRPC server so that clients can actually use our service. The following snippet shows how we do this for our `RouteGuide` service:

​	实现所有方法后，我们还需要启动一个 gRPC 服务器，以便客户端实际使用我们的服务。以下代码段展示了我们如何为 `RouteGuide` 服务执行此操作：

```js
function getServer() {
  var server = new grpc.Server();
  server.addService(routeguide.RouteGuide.service, {
    getFeature: getFeature,
    listFeatures: listFeatures,
    recordRoute: recordRoute,
    routeChat: routeChat
  });
  return server;
}
var routeServer = getServer();
routeServer.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {
  routeServer.start();
});
```

As you can see, we build and start our server with the following steps:

​	如您所见，我们通过以下步骤构建并启动服务器：

1. Create a `Server` constructor from the `RouteGuide` service descriptor.
   从 `RouteGuide` 服务描述符创建 `Server` 构造函数。
2. Implement the service methods.
   实现服务方法。
3. Create an instance of the server by calling the `Server` constructor with the method implementations.
   通过使用带有方法实现的 `Server` 构造函数调用来创建服务器实例。
4. Specify the address and port we want to use to listen for client requests using the instance’s `bind()` method.
   使用实例的 `bind()` 方法指定我们想要用来侦听客户端请求的地址和端口。
5. Call `start()` on the instance to start the RPC server.
   在实例上调用 `start()` 来启动 RPC 服务器。

### Creating the client 创建客户端

In this section, we’ll look at creating a Node.js client for our `RouteGuide` service. You can see our complete example client code in [examples/routeguide/dynamic_codegen/route_guide_client.js](https://github.com/grpc/grpc-node/blob/@grpc/grpc-js@1.9.0/examples/routeguide/dynamic_codegen/route_guide_client.js).

​	在本节中，我们将研究如何为我们的 `RouteGuide` 服务创建 Node.js 客户端。您可以在 examples/routeguide/dynamic_codegen/route_guide_client.js 中看到我们完整的示例客户端代码。

#### Creating a stub 创建存根

To call service methods, we first need to create a *stub*. To do this, we just need to call the RouteGuide stub constructor, specifying the server address and port.

​	要调用服务方法，我们首先需要创建一个存根。为此，我们只需要调用 RouteGuide 存根构造函数，并指定服务器地址和端口。

```js
new routeguide.RouteGuide('localhost:50051', grpc.credentials.createInsecure());
```

#### Calling service methods 调用服务方法

Now let’s look at how we call our service methods. Note that all of these methods are asynchronous: they use either events or callbacks to retrieve results.

​	现在让我们看看如何调用我们的服务方法。请注意，所有这些方法都是异步的：它们使用事件或回调来检索结果。

##### Simple RPC 简单 RPC

Calling the simple RPC `GetFeature` is nearly as straightforward as calling a local asynchronous method.

​	调用简单的 RPC `GetFeature` 与调用本地异步方法几乎一样简单。

```js
var point = {latitude: 409146138, longitude: -746188906};
stub.getFeature(point, function(err, feature) {
  if (err) {
    // process error
  } else {
    // process feature
  }
});
```

As you can see, we create and populate a request object. Finally, we call the method on the stub, passing it the request and callback. If there is no error, then we can read the response information from the server from our response object.

​	正如您所见，我们创建并填充了一个请求对象。最后，我们调用存根上的方法，将请求和回调传递给它。如果没有错误，那么我们就可以从我们的响应对象中读取服务器的响应信息。

```js
console.log('Found feature called "' + feature.name + '" at ' +
    feature.location.latitude/COORD_FACTOR + ', ' +
    feature.location.longitude/COORD_FACTOR);
```

##### Streaming RPCs 流式 RPC

Now let’s look at our streaming methods. If you’ve already read [Creating the server]({{< ref "/Languages/Node/Basicstutorial#server" >}}) some of this may look very familiar - streaming RPCs are implemented in a similar way on both sides. Here’s where we call the server-side streaming method `ListFeatures`, which returns a stream of geographical `Feature`s:

​	现在让我们看看我们的流式方法。如果您已经阅读了创建服务器，其中一些可能看起来非常熟悉 - 流式 RPC 在双方以类似的方式实现。我们在这里调用服务器端流式方法 `ListFeatures` ，它返回一个地理 `Feature` 流：

```js
var call = client.listFeatures(rectangle);
  call.on('data', function(feature) {
      console.log('Found feature called "' + feature.name + '" at ' +
          feature.location.latitude/COORD_FACTOR + ', ' +
          feature.location.longitude/COORD_FACTOR);
  });
  call.on('end', function() {
    // The server has finished sending
  });
  call.on('error', function(e) {
    // An error has occurred and the stream has been closed.
  });
  call.on('status', function(status) {
    // process status
  });
```

Instead of passing the method a request and callback, we pass it a request and get a `Readable` stream object back. The client can use the `Readable`’s `'data'` event to read the server’s responses. This event fires with each `Feature` message object until there are no more messages. Errors in the `'data'` callback will not cause the stream to be closed. The `'error'` event indicates that an error has occurred and the stream has been closed. The `'end'` event indicates that the server has finished sending and no errors occurred. Only one of `'error'` or `'end'` will be emitted. Finally, the `'status'` event fires when the server sends the status.

​	我们不是向该方法传递请求和回调，而是向它传递请求并获取 `Readable` 流对象作为返回值。客户端可以使用 `Readable` 的 `'data'` 事件来读取服务器的响应。此事件会随着每个 `Feature` 消息对象触发，直到没有更多消息为止。 `'data'` 回调中的错误不会导致流关闭。 `'error'` 事件表示已发生错误并且流已关闭。 `'end'` 事件表示服务器已完成发送且未发生错误。只会发出 `'error'` 或 `'end'` 中的一个。最后，当服务器发送状态时， `'status'` 事件触发。

The client-side streaming method `RecordRoute` is similar, except there we pass the method a callback and get back a `Writable`.

​	客户端流方法 `RecordRoute` 类似，不同之处在于我们向该方法传递回调并获取 `Writable` 作为返回值。

```js
var call = client.recordRoute(function(error, stats) {
  if (error) {
    callback(error);
  }
  console.log('Finished trip with', stats.point_count, 'points');
  console.log('Passed', stats.feature_count, 'features');
  console.log('Travelled', stats.distance, 'meters');
  console.log('It took', stats.elapsed_time, 'seconds');
});
function pointSender(lat, lng) {
  return function(callback) {
    console.log('Visiting point ' + lat/COORD_FACTOR + ', ' +
        lng/COORD_FACTOR);
    call.write({
      latitude: lat,
      longitude: lng
    });
    _.delay(callback, _.random(500, 1500));
  };
}
var point_senders = [];
for (var i = 0; i < num_points; i++) {
  var rand_point = feature_list[_.random(0, feature_list.length - 1)];
  point_senders[i] = pointSender(rand_point.location.latitude,
                                 rand_point.location.longitude);
}
async.series(point_senders, function() {
  call.end();
});
```

Once we’ve finished writing our client’s requests to the stream using `write()`, we need to call `end()` on the stream to let gRPC know that we’ve finished writing. If the status is `OK`, the `stats` object will be populated with the server’s response.

​	一旦我们使用 `write()` 完成向流中写入客户端请求，我们需要在流上调用 `end()` ，以让 gRPC 知道我们已完成写入。如果状态是 `OK` ，则 `stats` 对象将填充服务器的响应。

Finally, let’s look at our bidirectional streaming RPC `routeChat()`. In this case, we just pass a context to the method and get back a `Duplex` stream object, which we can use to both write and read messages.

​	最后，我们来看一下双向流式 RPC `routeChat()` 。在这种情况下，我们只需将上下文传递给该方法，然后返回 `Duplex` 流对象，我们可以使用该对象来写入和读取消息。

```js
var call = client.routeChat();
```

The syntax for reading and writing here is exactly the same as for our client-streaming and server-streaming methods. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.

​	此处用于读取和写入的语法与我们的客户端流式传输和服务器流式传输方法完全相同。尽管每一方总是会按写入顺序收到另一方的消息，但客户端和服务器都可以按任何顺序读取和写入——流完全独立地运行。

### Try it out! 试一试！

Build the client and server:

​	构建客户端和服务器：

```sh
$ npm install
```

Run the server: 
​	运行服务器：

```sh
$ node ./routeguide/dynamic_codegen/route_guide_server.js --db_path=./routeguide/dynamic_codegen/route_guide_db.json
```

From a different terminal, run the client:

​	从另一个终端运行客户端：

```sh
$ node ./routeguide/dynamic_codegen/route_guide_client.js --db_path=./routeguide/dynamic_codegen/route_guide_db.json
```

