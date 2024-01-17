+++
title = "Generated code"
date = 2024-01-17T08:51:13+08:00
weight = 30
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/python/generated-code/](https://grpc.io/docs/languages/python/generated-code/)

# Generated-code reference 生成代码参考





gRPC Python relies on the protocol buffers compiler (`protoc`) to generate code. It uses a plugin to supplement the generated code by plain `protoc` with gRPC-specific code. For a `.proto` service description containing gRPC services, the plain `protoc` generated code is synthesized in a `_pb2.py` file, and the gRPC-specific code lands in a `_pb2_grpc.py` file. The latter python module imports the former. The focus of this page is on the gRPC-specific subset of the generated code.

​	gRPC Python 依赖于协议缓冲区编译器 ( `protoc` ) 来生成代码。它使用插件来补充生成的代码，用纯 `protoc` 和 gRPC 特定的代码。对于包含 gRPC 服务的 `.proto` 服务描述，纯 `protoc` 生成的代码在 `_pb2.py` 文件中合成，而 gRPC 特定的代码位于 `_pb2_grpc.py` 文件中。后一个 python 模块导入前一个。此页面的重点是生成的代码中特定于 gRPC 的子集。

## Example 示例

Consider the following `FortuneTeller` proto service:

​	考虑以下 `FortuneTeller` proto 服务：

```proto
service FortuneTeller {
  // Returns the horoscope and zodiac sign for the given month and day.
  rpc TellFortune(HoroscopeRequest) returns (HoroscopeResponse) {
    // errors: invalid month or day, fortune unavailable
  }

  // Replaces the fortune for the given zodiac sign with the provided one.
  rpc SuggestFortune(SuggestionRequest) returns (SuggestionResponse) {
    // errors: invalid zodiac sign
  }
}
```

When the service is compiled, the gRPC `protoc` plugin generates code similar to the following `_pb2_grpc.py` file:

​	当编译服务时，gRPC `protoc` 插件会生成类似于以下 `_pb2_grpc.py` 文件的代码：

```python
import grpc

import fortune_pb2

class FortuneTellerStub(object):

  def __init__(self, channel):
    """Constructor.

    Args:
      channel: A grpc.Channel.
    """
    self.TellFortune = channel.unary_unary(
        '/example.FortuneTeller/TellFortune',
        request_serializer=fortune_pb2.HoroscopeRequest.SerializeToString,
        response_deserializer=fortune_pb2.HoroscopeResponse.FromString,
        )
    self.SuggestFortune = channel.unary_unary(
        '/example.FortuneTeller/SuggestFortune',
        request_serializer=fortune_pb2.SuggestionRequest.SerializeToString,
        response_deserializer=fortune_pb2.SuggestionResponse.FromString,
        )


class FortuneTellerServicer(object):

  def TellFortune(self, request, context):
    """Returns the horoscope and zodiac sign for the given month and day.
    errors: invalid month or day, fortune unavailable
    """
    context.set_code(grpc.StatusCode.UNIMPLEMENTED)
    context.set_details('Method not implemented!')
    raise NotImplementedError('Method not implemented!')

  def SuggestFortune(self, request, context):
    """Replaces the fortune for the given zodiac sign with the provided
one.
    errors: invalid zodiac sign
    """
    context.set_code(grpc.StatusCode.UNIMPLEMENTED)
    context.set_details('Method not implemented!')
    raise NotImplementedError('Method not implemented!')


def add_FortuneTellerServicer_to_server(servicer, server):
  rpc_method_handlers = {
      'TellFortune': grpc.unary_unary_rpc_method_handler(
          servicer.TellFortune,
          request_deserializer=fortune_pb2.HoroscopeRequest.FromString,
          response_serializer=fortune_pb2.HoroscopeResponse.SerializeToString,
      ),
      'SuggestFortune': grpc.unary_unary_rpc_method_handler(
          servicer.SuggestFortune,
          request_deserializer=fortune_pb2.SuggestionRequest.FromString,
          response_serializer=fortune_pb2.SuggestionResponse.SerializeToString,
      ),
  }
  generic_handler = grpc.method_handlers_generic_handler(
      'example.FortuneTeller', rpc_method_handlers)
  server.add_generic_rpc_handlers((generic_handler,))
```

## Code Elements 代码元素

The gRPC generated code starts by importing the `grpc` package and the plain `_pb2` module, synthesized by `protoc`, which defines non-gRPC-specific code elements, like the classes corresponding to protocol buffers messages and descriptors used by reflection.

​	gRPC 生成的代码首先导入 `grpc` 包和 `_pb2` 模块，由 `protoc` 合成，它定义了非 gRPC 特定的代码元素，例如与协议缓冲区消息和反射使用的描述符相对应的类。

For each service `Foo` in the `.proto` file, three primary elements are generated:

​	对于 `.proto` 文件中的每个服务 `Foo` ，会生成三个主要元素：

- [**Stub**]({{< ref "/Languages/Python/Generatedcode#stub" >}}): `FooStub` used by the client to connect to a gRPC service.
  存根： `FooStub` 由客户端用于连接到 gRPC 服务。
- [**Servicer**]({{< ref "/Languages/Python/Generatedcode#servicer" >}}): `FooServicer` used by the server to implement a gRPC service.
  服务员： `FooServicer` 由服务器用于实现 gRPC 服务。
- [**Registration Function**]({{< ref "/Languages/Python/Generatedcode#registration-function" >}}): `add_FooServicer_to_server` function used to register a servicer with a `grpc.Server` object.
  注册函数： `add_FooServicer_to_server` 用于将服务员注册到 `grpc.Server` 对象的函数。

### Stub 存根

The generated `Stub` class is used by the gRPC clients. It has a constructor that takes a `grpc.Channel` object and initializes the stub. For each method in the service, the initializer adds a corresponding attribute to the stub object with the same name. Depending on the RPC type (unary or streaming), the value of that attribute will be callable objects of type [UnaryUnaryMultiCallable](https://grpc.io/grpc/python/grpc.html?#grpc.UnaryUnaryMultiCallable), [UnaryStreamMultiCallable](https://grpc.io/grpc/python/grpc.html?#grpc.UnaryStreamMultiCallable), [StreamUnaryMultiCallable](https://grpc.io/grpc/python/grpc.html?#grpc.StreamUnaryMultiCallable), or [StreamStreamMultiCallable](https://grpc.io/grpc/python/grpc.html?#grpc.StreamStreamMultiCallable).

​	生成的 `Stub` 类由 gRPC 客户端使用。它有一个构造函数，该函数采用 `grpc.Channel` 对象并初始化存根。对于服务中的每个方法，初始化程序都会向存根对象添加一个具有相同名称的相应属性。根据 RPC 类型（单播或流式），该属性的值将是类型为 UnaryUnaryMultiCallable、UnaryStreamMultiCallable、StreamUnaryMultiCallable 或 StreamStreamMultiCallable 的可调用对象。

### Servicer 服务员

For each service, a `Servicer` class is generated, which serves as the superclass of a service implementation. For each method in the service, a corresponding function in the `Servicer` class is generated. Override this function with the service implementation. Comments associated with code elements in the `.proto` file appear as docstrings in the generated python code.

​	对于每项服务，都会生成一个 `Servicer` 类，该类用作服务实现的超类。对于服务中的每个方法，都会在 `Servicer` 类中生成一个相应的功能。使用服务实现覆盖此功能。与 `.proto` 文件中的代码元素关联的注释显示为生成 Python 代码中的文档字符串。

### Registration Function 注册功能

For each service, a function is generated that registers a `Servicer` object implementing it on a `grpc.Server` object, so that the server can route queries to the respective servicer. This function takes an object that implements the `Servicer`, typically an instance of a subclass of the generated `Servicer` code element described above, and a [grpc.Server](https://grpc.io/grpc/python/_modules/grpc.html#Server) object.

​	对于每个服务，都会生成一个函数，该函数注册一个在 `grpc.Server` 对象上实现它的 `Servicer` 对象，以便服务器可以将查询路由到相应的服务提供者。此函数采用一个实现 `Servicer` 的对象，通常是上面描述的生成的 `Servicer` 代码元素的子类的实例，以及一个 grpc.Server 对象。
