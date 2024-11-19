+++
title = "Quick start"
date = 2024-11-19T10:19:42+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/languages/java/quickstart/](https://grpc.io/docs/languages/java/quickstart/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Quick start

This guide gets you started with gRPC in Java with a simple working example.



### Prerequisites

- [JDK](https://jdk.java.net/) version 7 or higher

### Get the example code

The example code is part of the [grpc-java](https://github.com/grpc/grpc-java) repo.

1. [Download the repo as a zip file](https://github.com/grpc/grpc-java/archive/v1.68.1.zip) and unzip it, or clone the repo:

   ```sh
   $ git clone -b v1.68.1 --depth 1 https://github.com/grpc/grpc-java
   ```

2. Change to the examples directory:

   ```sh
   $ cd grpc-java/examples
   ```

### Run the example

From the `examples` directory:

1. Compile the client and server

   ```sh
   $ ./gradlew installDist
   ```

2. Run the server:

   ```sh
   $ ./build/install/examples/bin/hello-world-server
   INFO: Server started, listening on 50051
   ```

3. From another terminal, run the client:

   ```sh
   $ ./build/install/examples/bin/hello-world-client
   INFO: Will try to greet world ...
   INFO: Greeting: Hello world
   ```

Congratulations! You’ve just run a client-server application with gRPC.

#### Note

We’ve omitted timestamps from the client and server trace output shown in this page.

### Update the gRPC service

In this section you’ll update the application by adding an extra server method. The gRPC service is defined using [protocol buffers](https://developers.google.com/protocol-buffers). To learn more about how to define a service in a `.proto` file see [Basics tutorial](https://grpc.io/docs/languages/java/basics/). For now, all you need to know is that both the server and the client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

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

Open `src/main/proto/helloworld.proto` and add a new `SayHelloAgain()` method with the same request and response types as `SayHello()`:

```protobuf
// The greeting service definition.
service Greeter {
  // Sends a greeting. Original method.
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // Sends another greeting. New method.
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  // The name of the user.
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  // The greeting message.
  string message = 1;
}
```

Remember to save the file!

### Update the app

When you build the example, the build process regenerates `GreeterGrpc.java`, which contains the generated gRPC client and server classes. This also regenerates classes for populating, serializing, and retrieving our request and response types.

However, you still need to implement and call the new method in the hand-written parts of the example app.

#### Update the server

In the same directory, open `src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java`. Implement the new method like this:

```java
// Implementation of the gRPC service on the server-side.
private class GreeterImpl extends GreeterGrpc.GreeterImplBase {

  @Override
  public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
    // Generate a greeting message for the original method
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello " + req.getName()).build();

    // Send the reply back to the client.
    responseObserver.onNext(reply);

    // Indicate that no further messages will be sent to the client.
    responseObserver.onCompleted();
  }

  @Override
  public void sayHelloAgain(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
    // Generate another greeting message for the new method.
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello again " + req.getName()).build();

    // Send the reply back to the client.
    responseObserver.onNext(reply);

    // Indicate that no further messages will be sent to the client.
    responseObserver.onCompleted();
  }
}
```

#### Update the client

In the same directory, open `src/main/java/io/grpc/examples/helloworld/HelloWorldClient.java`. Call the new method like this:

```java
// Client-side logic for interacting with the gRPC service.
public void greet(String name) {
  // Log a message indicating the intention to greet a user.
  logger.info("Will try to greet " + name + " ...");

  // Creating a request with the user's name.
  HelloRequest request = HelloRequest.newBuilder().setName(name).build();
  HelloReply response;
  try {
    // Call the original method on the server.
    response = blockingStub.sayHello(request);
  } catch (StatusRuntimeException e) {
    // Log a warning if the RPC fails.
    logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
    return;
  }

  // Log the response from the original method.
  logger.info("Greeting: " + response.getMessage());

  try {
    // Call the new method on the server.
    response = blockingStub.sayHelloAgain(request);
  } catch (StatusRuntimeException e) {
    // Log a warning if the RPC fails.
    logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
    return;
  }

  // Log the response from the new method.
  logger.info("Greeting: " + response.getMessage());
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
   $ ./build/install/examples/bin/hello-world-server
   INFO: Server started, listening on 50051
   ```

3. From another terminal, run the client:

   ```sh
   $ ./build/install/examples/bin/hello-world-client
   INFO: Will try to greet world ...
   INFO: Greeting: Hello world
   INFO: Greeting: Hello again world
   ```

### What’s next

- Learn how gRPC works in [Introduction to gRPC](https://grpc.io/docs/what-is-grpc/introduction/) and [Core concepts](https://grpc.io/docs/what-is-grpc/core-concepts/).
- Work through the [Basics tutorial](https://grpc.io/docs/languages/java/basics/).
- Explore the [API reference](https://grpc.io/docs/languages/java/api).
