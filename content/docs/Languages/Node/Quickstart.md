+++
title = "Quick start"
date = 2024-11-19T10:19:42+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/languages/node/quickstart/](https://grpc.io/docs/languages/node/quickstart/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Quick start

This guide gets you started with gRPC in Node with a simple working example.



### Prerequisites

- Node version 8.13.0 or higher

### Download the example

You’ll need a local copy of the example code to work through this quick start. Download the example code from our GitHub repository (the following command clones the entire repository, but you just need the examples for this quick start and other tutorials):

```sh
# Clone the repository to get the example code
$ git clone -b @grpc/grpc-js@1.9.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc-node
# Navigate to the node example
$ cd grpc-node/examples
# Install the example's dependencies
$ npm install
# Navigate to the dynamic codegen "hello, world" Node example:
$ cd helloworld/dynamic_codegen
```

### Run a gRPC application

From the `examples/helloworld/dynamic_codegen` directory:

1. Run the server:

   ```sh
   $ node greeter_server.js
   ```

2. From another terminal, run the client:

   ```sh
   $ node greeter_client.js
   ```

Congratulations! You’ve just run a client-server application with gRPC.

### Update the gRPC service

Now let’s look at how to update the application with an extra method on the server for the client to call. Our gRPC service is defined using protocol buffers; you can find out lots more about how to define a service in a `.proto` file in [Basics tutorial]({{< ref "/docs/Languages/Node/Basicstutorial" >}}). For now all you need to know is that both the server and the client “stub” have a `SayHello` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that this method is defined like this:

```proto
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

Let’s update this so that the `Greeter` service has two methods. Edit `examples/protos/helloworld.proto` and update it with a new `SayHelloAgain` method, with the same request and response types:

```proto
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

### Update and run the application

We now have a new service definition, but we still need to implement and call the new method in the human-written parts of our example application.

#### Update the server

In the same directory, open `greeter_server.js`. Implement the new method like this:

```js
function sayHello(call, callback) {
  callback(null, {message: 'Hello ' + call.request.name});
}

function sayHelloAgain(call, callback) {
  callback(null, {message: 'Hello again, ' + call.request.name});
}

function main() {
  var server = new grpc.Server();
  server.addService(hello_proto.Greeter.service,
                         {sayHello: sayHello, sayHelloAgain: sayHelloAgain});
  server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {
    server.start();
  });
}
```

#### Update the client

In the same directory, open `greeter_client.js`. Call the new method like this:

```js
function main() {
  var client = new hello_proto.Greeter('localhost:50051',
                                       grpc.credentials.createInsecure());
  client.sayHello({name: 'you'}, function(err, response) {
    console.log('Greeting:', response.message);
  });
  client.sayHelloAgain({name: 'you'}, function(err, response) {
    console.log('Greeting:', response.message);
  });
}
```

#### Run!

Just like we did before, from the `examples/helloworld/dynamic_codegen` directory:

1. Run the server:

   ```sh
   $ node greeter_server.js
   ```

2. From another terminal, run the client:

   ```sh
   $ node greeter_client.js
   ```

### What’s next

- Learn how gRPC works in [Introduction to gRPC]({{< ref "/docs/WhatisgRPC/Introduction" >}}) and [Core concepts]({{< ref "/docs/WhatisgRPC/Coreconcepts" >}}).
- Work through the [Basics tutorial]({{< ref "/docs/Languages/Node/Basicstutorial" >}}).
- Explore the [API reference]({{< ref "/docs/Languages/Node/API" >}}).
- We have more than one grpc implementation for Node. For the pros and cons of each package, see this [package feature comparison](https://github.com/grpc/grpc-node/blob/master/PACKAGE-COMPARISON.md).
