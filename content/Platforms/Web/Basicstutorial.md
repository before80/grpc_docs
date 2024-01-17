+++
title = "Basics tutorial"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/platforms/web/basics/](https://grpc.io/docs/platforms/web/basics/)

# Basics tutorial 基础教程

A basic tutorial introduction to gRPC-web.

​	gRPC-web 基本教程简介。



This tutorial provides a basic introduction on how to use gRPC-Web from browsers.

​	本教程提供有关如何从浏览器使用 gRPC-Web 的基本介绍。

By walking through this example you’ll learn how to:

​	通过本示例，您将学习如何执行以下操作：

- Define a service in a .proto file.
  在 .proto 文件中定义服务。
- Generate client code using the protocol buffer compiler.
  使用协议缓冲区编译器生成客户端代码。
- Use the gRPC-Web API to write a simple client for your service.
  使用 gRPC-Web API 为您的服务编写一个简单的客户端。

It assumes a passing familiarity with [protocol buffers](https://protobuf.dev/overview).

​	它假定您对协议缓冲区有一定的了解。

### Why use gRPC and gRPC-Web? 为什么使用 gRPC 和 gRPC-Web？

With gRPC you can define your service once in a .proto file and implement clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet - all the complexity of communication between different languages and environments is handled for you by gRPC. You also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface updating. gRPC-Web lets you access gRPC services built in this manner from browsers using an idiomatic API.

​	使用 gRPC，您可以在 .proto 文件中一次定义您的服务，并在 gRPC 支持的任何语言中实现客户端和服务器，进而可以在从大型数据中心内的服务器到您自己的平板电脑等各种环境中运行 - gRPC 会为您处理不同语言和环境之间通信的所有复杂性。您还可以获得使用协议缓冲区的所有优势，包括高效序列化、简单的 IDL 和轻松的接口更新。gRPC-Web 允许您使用习惯用语 API 从浏览器访问以这种方式构建的 gRPC 服务。

### Define the Service 定义服务

The first step when creating a gRPC service is to define the service methods and their request and response message types using protocol buffers. In this example, we define our `EchoService` in a file called [`echo.proto`](https://github.com/grpc/grpc-web/blob/0.4.0/net/grpc/gateway/examples/echo/echo.proto). For more information about protocol buffers and proto3 syntax, please see the [protobuf documentation](https://protobuf.dev/).

​	创建 gRPC 服务时的第一步是使用协议缓冲区定义服务方法及其请求和响应消息类型。在此示例中，我们在名为 `EchoService` 的文件中定义我们的 `echo.proto` 。有关协议缓冲区和 proto3 语法的更多信息，请参阅 protobuf 文档。

```protobuf
message EchoRequest {
  string message = 1;
}

message EchoResponse {
  string message = 1;
}

service EchoService {
  rpc Echo(EchoRequest) returns (EchoResponse);
}
```

### Implement gRPC Backend Server 实现 gRPC 后端服务器

Next, we implement our EchoService interface using Node in the backend gRPC `EchoServer`. This will handle requests from clients. See the file [`node-server/server.js`](https://github.com/grpc/grpc-web/blob/master/net/grpc/gateway/examples/echo/node-server/server.js) for details.

​	接下来，我们在后端 gRPC `EchoServer` 中使用 Node 实现我们的 EchoService 接口。这将处理来自客户端的请求。有关详细信息，请参阅文件 `node-server/server.js` 。

You can implement the server in any language supported by gRPC. Please see the [main page](https://grpc.io/docs/) for more details.

​	您可以在 gRPC 支持的任何语言中实现服务器。有关更多详细信息，请参阅主页。

```js
function doEcho(call, callback) {
  callback(null, {message: call.request.message});
}
```

### Configure the Envoy Proxy 配置 Envoy 代理

In this example, we will use the [Envoy](https://www.envoyproxy.io/) proxy to forward the gRPC browser request to the backend server. You can see the complete config file in [envoy.yaml](https://github.com/grpc/grpc-web/blob/master/net/grpc/gateway/examples/echo/envoy.yaml)

​	在此示例中，我们将使用 Envoy 代理将 gRPC 浏览器请求转发到后端服务器。您可以在 envoy.yaml 中看到完整的配置文件

To forward the gRPC requests to the backend server, we need a block like this:

​	要将 gRPC 请求转发到后端服务器，我们需要类似这样的块：

```yaml
admin:
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: echo_service }
          http_filters:
          - name: envoy.grpc_web
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.grpc_web.v3.GrpcWeb
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
  clusters:
  - name: echo_service
    connect_timeout: 0.25s
    type: LOGICAL_DNS
    typed_extension_protocol_options:
      envoy.extensions.upstreams.http.v3.HttpProtocolOptions:
        "@type": type.googleapis.com/envoy.extensions.upstreams.http.v3.HttpProtocolOptions
        explicit_http_config:
          http2_protocol_options: {}
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: echo_service
      endpoints:
        - lb_endpoints:
          - endpoint:
              address:
                socket_address:
                  address: node-server
                  port_value: 9090
```

You may also need to add some CORS setup to make sure the browser can request cross-origin content.

​	您可能还需要添加一些 CORS 设置，以确保浏览器可以请求跨域内容。

In this simple example, the browser makes gRPC requests to port `:8080`. Envoy forwards the request to the backend gRPC server listening on port `:9090`.

​	在此简单示例中，浏览器向端口 `:8080` 发出 gRPC 请求。Envoy 将请求转发到侦听端口 `:9090` 的后端 gRPC 服务器。

### Generate Protobuf Messages and Service Client Stub 生成 Protobuf 消息和服务客户端存根

To generate the protobuf message classes from our `echo.proto`, run the following command:

​	要从我们的 `echo.proto` 生成 protobuf 消息类，请运行以下命令：

```sh
$ protoc -I=$DIR echo.proto \
  --js_out=import_style=commonjs:$OUT_DIR
```

The `import_style` option passed to the `--js_out` flag makes sure the generated files will have CommonJS style `require()` statements.

​	传递给 `--js_out` 标志的 `import_style` 选项可确保生成的文件具有 CommonJS 样式 `require()` 语句。

To generate the gRPC-Web service client stub, first you need the gRPC-Web protoc plugin. To compile the plugin `protoc-gen-grpc-web`, you need to run this from the repo’s root directory:

​	要生成 gRPC-Web 服务客户端存根，首先需要 gRPC-Web protoc 插件。要编译插件 `protoc-gen-grpc-web` ，您需要从存储库的根目录运行以下命令：

```sh
$ cd grpc-web
$ sudo make install-plugin
```

To generate the service client stub file, run this command:

​	要生成服务客户端存根文件，请运行以下命令：

```sh
$ protoc -I=$DIR echo.proto \
  --grpc-web_out=import_style=commonjs,mode=grpcwebtext:$OUT_DIR
```

In the `--grpc-web_out` param above:

​	在上面的 `--grpc-web_out` 参数中：

- `mode` can be `grpcwebtext` (default) or `grpcweb`
  `mode` 可以是 `grpcwebtext` （默认）或 `grpcweb`
- `import_style` can be `closure` (default) or `commonjs`
  `import_style` 可以是 `closure` （默认）或 `commonjs`

Our command generates the client stub, by default, to the file `echo_grpc_web_pb.js`.

​	我们的命令会将客户端存根（默认情况下）生成到文件 `echo_grpc_web_pb.js` 。

### Write JS Client Code 编写 JS 客户端代码

Now you are ready to write some JS client code. Put this in a `client.js` file.

​	现在，您可以编写一些 JS 客户端代码了。将其放入 `client.js` 文件中。

```js
const {EchoRequest, EchoResponse} = require('./echo_pb.js');
const {EchoServiceClient} = require('./echo_grpc_web_pb.js');

var echoService = new EchoServiceClient('http://localhost:8080');

var request = new EchoRequest();
request.setMessage('Hello World!');

echoService.echo(request, {}, function(err, response) {
  // ...
});
```

You will need a `package.json` file

​	您将需要一个 `package.json` 文件

```json
{
  "name": "grpc-web-commonjs-example",
  "dependencies": {
    "google-protobuf": "^3.6.1",
    "grpc-web": "^0.4.0"
  },
  "devDependencies": {
    "browserify": "^16.2.2",
    "webpack": "^4.16.5",
    "webpack-cli": "^3.1.0"
  }
}
```

### Compile the JS Library 编译 JS 库

Finally, putting all these together, we can compile all the relevant JS files into one single JS library that can be used in the browser.

​	最后，将所有这些内容放在一起，我们可以将所有相关的 JS 文件编译成一个可以在浏览器中使用的单个 JS 库。

```sh
$ npm install
$ npx webpack client.js
```

Now embed `dist/main.js` into your project and see it in action!

​	现在将 `dist/main.js` 嵌入到您的项目中，并查看它的实际效果！
