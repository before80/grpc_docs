+++
title = "Quick start"
date = 2024-11-19T10:19:42+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/languages/kotlin/quickstart/](https://grpc.io/docs/languages/kotlin/quickstart/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Quick start

This guide gets you started with gRPC in Kotlin with a simple working example.



### Prerequisites

- [Kotlin](https://kotlinlang.org/) version 1.3 or higher
- [JDK](https://jdk.java.net/) version 7 or higher

### Get the example code

The example code is part of the [grpc-kotlin](https://github.com/grpc/grpc-kotlin) repo.

1. [Download the repo as a zip file](https://github.com/grpc/grpc-kotlin/archive/master.zip) and unzip it, or clone the repo:

   ```sh
   $ git clone --depth 1 https://github.com/grpc/grpc-kotlin
   ```

2. Change to the examples directory:

   ```sh
   $ cd grpc-kotlin/examples
   ```

### Run the example

From the `examples` directory:

1. Compile the client and server

   ```sh
   $ ./gradlew installDist
   ```

2. Run the server:

   ```sh
   $ ./server/build/install/server/bin/hello-world-server
   Server started, listening on 50051
   ```

3. From another terminal, run the client:

   ```sh
   $ ./client/build/install/client/bin/hello-world-client
   Received: Hello world
   ```

Congratulations! You’ve just run a client-server app with gRPC.

### Update the gRPC service

In this section, you’ll update the app with an extra server method. The app’s gRPC service, named `Greeter`, is defined using [protocol buffers](https://developers.google.com/protocol-buffers). To learn more about how to define a service in a `.proto` file, see [Basics tutorial]({{< ref "/docs/Languages/Kotlin/Basicstutorial" >}}). For now, all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

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

### Update the app

When you build the example, the build process regenerates `HelloWorldProtoGrpcKt.kt`, which contains the generated gRPC client and server classes. This also regenerates classes for populating, serializing, and retrieving our request and response types.

However, you still need to implement and call the new method in the hand-written parts of the example app.

#### Update the server

Open `helloworld/HelloWorldServer.kt` from the [server/src/main/kotlin/io/grpc/examples](https://github.com/grpc/grpc-kotlin/tree/master/examples/server/src/main/kotlin/io/grpc/examples) folder. Implement the new method like this:

```kotlin
private class HelloWorldService : GreeterGrpcKt.GreeterCoroutineImplBase() {
  override suspend fun sayHello(request: HelloRequest) = helloReply {
    message = "Hello ${request.name}"
  }

  override suspend fun sayHelloAgain(request: HelloRequest) = helloReply {
    message = "Hello again ${request.name}"
  }
}
```

#### Update the client

Open `helloworld/HelloWorldClient.kt` from the [client/src/main/kotlin/io/grpc/examples](https://github.com/grpc/grpc-kotlin/tree/master/examples/client/src/main/kotlin/io/grpc/examples) folder. Call the new method like this:

```kotlin
class HelloWorldClient(
    private val channel: ManagedChannel
) : Closeable {
  private val stub: GreeterCoroutineStub = GreeterCoroutineStub(channel)

  suspend fun greet(name: String) {
    val request = helloRequest { this.name = name }
    val response = stub.sayHello(request)
    println("Received: ${response.message}")
    val againResponse = stub.sayHelloAgain(request)
    println("Received: ${againResponse.message}")
  }

  override fun close() {
    channel.shutdown().awaitTermination(5, TimeUnit.SECONDS)
  }
}
```

### Run the updated app

Run the client and server like you did before. Execute the following commands from the `examples` directory:

1. Compile the client and server:

   ```sh
   $ ./gradlew installDist
   ```

2. Run the server:

   ```sh
   $ ./server/build/install/server/bin/hello-world-server
   Server started, listening on 50051
   ```

3. From another terminal, run the client. This time, add a name as a command-line argument:

   ```sh
   $ ./client/build/install/client/bin/hello-world-client Alice
   Received: Hello Alice
   Received: Hello again Alice
   ```

### What’s next

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/docs/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/docs/WhatisgRPC/Coreconcepts" >}}).
- Work through the [Basics tutorial]({{< ref "/docs/Languages/Kotlin/Basicstutorial" >}}).
- Explore the [API reference]({{< ref "/docs/Languages/Kotlin/API" >}}).
