+++
title = "Quick start"
date = 2024-11-19T10:19:42+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/languages/php/quickstart/](https://grpc.io/docs/languages/php/quickstart/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Quick start

This guide gets you started with gRPC in PHP with a simple working example.



### Prerequisites

- PHP 7.0 or higher, PECL, Composer
- grpc extension, protocol buffers compiler: for installation instructions, see the [gRPC PHP readme](https://github.com/grpc/grpc/blob/v1.66.0/src/php/README.md).

### Get the example code

The example code is part of the [grpc](https://github.com/grpc/grpc) repo.

> Note
>
> You can only create gRPC clients in PHP. Use [another language](https://grpc.io/docs/languages/) to create a gRPC server.

1. Clone the [grpc](https://github.com/grpc/grpc) repo and its submodules:

   ```sh
   $ git clone --recurse-submodules -b v1.66.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc
   ```

2. Change to the quick start example directory:

   ```sh
   $ cd grpc/examples/php
   ```

3. Install the `grpc` composer package:

   ```sh
   $ ./greeter_proto_gen.sh
   $ composer install
   ```

### Run the example

1. Launch the quick start server: for example, follow the instructions given in the [Quick start for Node](https://grpc.io/docs/languages/node/quickstart/).

2. From the `examples/php` directory, run the PHP client:

   ```sh
   $ ./run_greeter_client.sh
   ```

Congratulations! You’ve just run a client-server application with gRPC.

### Update the gRPC service

Now let’s look at how to update the application with an extra method on the server for the client to call. Our gRPC service is defined using protocol buffers; you can find out lots more about how to define a service in a `.proto` file in [Basics tutorial](https://grpc.io/docs/languages/php/basics/). For now all you need to know is that both the server and the client “stub” have a `SayHello` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloResponse` from the server, and that this method is defined like this:

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

Let’s update this so that the `Greeter` service has two methods. Edit `examples/protos/helloworld.proto` and update it with a new `SayHelloAgain` method, with the same request and response types:

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

### Regenerate gRPC code

Next we need to update the gRPC code used by our application to use the new service definition. From the `grpc` root directory:

```sh
$ protoc --proto_path=examples/protos \
  --php_out=examples/php \
  --grpc_out=examples/php \
  --plugin=protoc-gen-grpc=bins/opt/grpc_php_plugin \
  ./examples/protos/helloworld.proto
```

or running the helper script under the `grpc/example/php` directory if you build grpc-php-plugin by source:

```sh
$ ./greeter_proto_gen.sh
```

This regenerates the protobuf files, which contain our generated client classes, as well as classes for populating, serializing, and retrieving our request and response types.

### Update and run the application

We now have new generated client code, but we still need to implement and call the new method in the human-written parts of our example application.

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
  server.addProtoService(hello_proto.Greeter.service,
                         {sayHello: sayHello, sayHelloAgain: sayHelloAgain});
  server.bind('0.0.0.0:50051', grpc.ServerCredentials.createInsecure());
  server.start();
}
...
```

#### Update the client

In the same directory, open `greeter_client.php`. Call the new method like this:

```php
$request = new Helloworld\HelloRequest();
$request->setName($name);
list($reply, $status) = $client->SayHello($request)->wait();
$message = $reply->getMessage();
list($reply, $status) = $client->SayHelloAgain($request)->wait();
$message = $reply->getMessage();
```

#### Run!

Just like we did before, from the `grpc-node/examples/helloworld/dynamic_codegen` directory:

1. Run the server:

   ```sh
   $ node greeter_server.js
   ```

2. From another terminal, from the `examples/php` directory, run the client:

   ```sh
   $ ./run_greeter_client.sh
   ```

### What’s next

- Learn how gRPC works in [Introduction to gRPC](https://grpc.io/docs/what-is-grpc/introduction/) and [Core concepts](https://grpc.io/docs/what-is-grpc/core-concepts/).
- Work through the [Basics tutorial](https://grpc.io/docs/languages/php/basics/).
- Explore the [API reference](https://grpc.io/docs/languages/php/api).
