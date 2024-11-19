+++
title = "Quick start"
date = 2024-11-19T10:19:42+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/languages/dart/quickstart/](https://grpc.io/docs/languages/dart/quickstart/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Quick start

This guide gets you started with gRPC in Dart with a simple working example.



### Prerequisites

- **[Dart](https://dart.dev/)** version 2.12 or higher, through the Dart or [Flutter](https://flutter.dev/) SDKs

  For installation instructions, see [Install Dart](https://dart.dev/install) or [Install Flutter](https://flutter.dev/docs/get-started/install).

- **[Protocol buffer](https://developers.google.com/protocol-buffers) compiler**, `protoc`, [version 3](https://protobuf.dev/programming-guides/proto3)

  For installation instructions, see [Protocol Buffer Compiler Installation](https://grpc.io/docs/protoc-installation/).

- **Dart plugin** for the protocol compiler:

  1. Install the protocol compiler plugin for Dart (`protoc-gen-dart`) using the following command:

     ```sh
     $ dart pub global activate protoc_plugin
     ```

  2. Update your `PATH` so that the `protoc` compiler can find the plugin:

     ```sh
     $ export PATH="$PATH:$HOME/.pub-cache/bin"
     ```

> Note
>
> Dart gRPC supports the Flutter and Server platforms.
>

### Get the example code

The example code is part of the [grpc-dart](https://github.com/grpc/grpc-dart) repo.

1. [Download the repo as a zip file](https://github.com/grpc/grpc-dart/archive/master.zip) and unzip it, or clone the repo:

   ```sh
   $ git clone https://github.com/grpc/grpc-dart
   ```

2. Change to the quick start example directory:

   ```sh
   $ cd grpc-dart/example/helloworld
   ```

### Run the example

From the `example/helloworld` directory:

1. Download package dependencies:

   ```sh
   $ dart pub get
   ```

2. Run the server:

   ```sh
   $ dart bin/server.dart
   ```

3. From another terminal, run the client:

   ```sh
   $ dart bin/client.dart
   Greeter client received: Hello, world!
   ```

Congratulations! You’ve just run a client-server application with gRPC.

### Update the app

In this section you’ll update the app to make use of an extra server method. The gRPC service is defined using [protocol buffers](https://developers.google.com/protocol-buffers). To learn more about how to define a service in a `.proto` file, see [Basics tutorial]({{< ref "/docs/Languages/Dart/Basicstutorial" >}}). For now, all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

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

#### Update the gRPC service

Open `protos/helloworld.proto` and add a new `SayHelloAgain()` method, with the same request and response types:

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

#### Regenerate gRPC code

Before you can use the new service method, you need to recompile the updated proto file. From the `example/helloworld` directory, run the following command:

```sh
$ protoc --dart_out=grpc:lib/src/generated -Iprotos protos/helloworld.proto
```

You’ll find the regenerated request and response classes, and client and server classes in the `lib/src/generated` directory.

Now implement and call the new RPC in the server and client code, respectively.

#### Update the server

Open `bin/server.dart` and add the following `sayHelloAgain()` method to the `GreeterService` class:

```dart
class GreeterService extends GreeterServiceBase {
  @override
  Future<HelloReply> sayHello(ServiceCall call, HelloRequest request) async {
    return HelloReply()..message = 'Hello, ${request.name}!';
  }

  @override
  Future<HelloReply> sayHelloAgain(ServiceCall call, HelloRequest request) async {
    return HelloReply()..message = 'Hello again, ${request.name}!';
  }
}
```

#### Update the client

Add a call to `sayHelloAgain()` in `bin/client.dart` like this:

```dart
Future<void> main(List<String> args) async {
  final channel = ClientChannel(
    'localhost',
    port: 50051,
    options: const ChannelOptions(credentials: ChannelCredentials.insecure()),
  );
  final stub = GreeterClient(channel);

  final name = args.isNotEmpty ? args[0] : 'world';

  try {
    var response = await stub.sayHello(HelloRequest()..name = name);
    print('Greeter client received: ${response.message}');
    response = await stub.sayHelloAgain(HelloRequest()..name = name);
    print('Greeter client received: ${response.message}');
  } catch (e) {
    print('Caught error: $e');
  }
  await channel.shutdown();
}
```

### Run the updated app

Run the client and server like you did before. Execute the following commands from the `example/helloworld` directory:

1. Run the server:

   ```sh
   $ dart bin/server.dart
   ```

2. From another terminal, run the client. This time, add a name as a command-line argument:

   ```sh
   $ dart bin/client.dart Alice
   ```

   You’ll see the following output:

   ```sh
   Greeter client received: Hello, Alice!
   Greeter client received: Hello again, Alice!
   ```

### Contributing

If you experience problems with Dart gRPC or have a feature request, [create an issue](https://github.com/grpc/grpc-dart/issues/new) over the [grpc-dart](https://github.com/grpc/grpc-dart) repo.

### What’s next

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/docs/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/docs/WhatisgRPC/Coreconcepts" >}}).
- Work through the [Basics tutorial]({{< ref "/docs/Languages/Dart/Basicstutorial" >}}).
- Explore the [API reference]({{< ref "/docs/Languages/Dart/API" >}}).
