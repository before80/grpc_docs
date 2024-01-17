+++
title = "Compression"
date = 2024-01-17T08:51:13+08:00
weight = 30
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/guides/compression/](https://grpc.io/docs/guides/compression/)

# Compression 压缩

How to compress the data sent over the wire while using gRPC.

​	使用 gRPC 时如何压缩通过网络发送的数据。



### Overview 概述

Compression is used to reduce the amount of bandwidth used when communicating between peers and can be enabled or disabled based on call or message level for all languages. For some languages, it is also possible to control compression settings at the channel level. Different languages also support different compression algorithms, including a customized compressor.

​	压缩用于减少对等方之间通信时使用的带宽，可以基于所有语言的呼叫或消息级别启用或禁用。对于某些语言，还可以控制通道级别的压缩设置。不同的语言还支持不同的压缩算法，包括自定义压缩器。

### Compression Method Asymmetry Between Peers 对等方之间的压缩方法不对称

gRPC allows asymmetrically compressed communication, whereby a response may be compressed differently with the request, or not compressed at all. A gRPC peer may choose to respond using a different compression method to that of the request, including not performing any compression, regardless of channel and RPC settings (for example, if compression would result in small or negative gains).

​	gRPC 允许不对称压缩通信，其中响应可能与请求不同地压缩，或者根本不压缩。gRPC 对等方可以选择使用不同于请求的压缩方法来响应，包括不执行任何压缩，而不管通道和 RPC 设置如何（例如，如果压缩会导致收益很小或为负）。

If a client message is compressed by an algorithm that is not supported by a server, the message will result in an `UNIMPLEMENTED` error status on the server. The server will include a `grpc-accept-encoding` header to the response which specifies the algorithms that the server accepts.

​	如果客户端消息由服务器不支持的算法压缩，则该消息将在服务器上导致 `UNIMPLEMENTED` 错误状态。服务器将在响应中包含 `grpc-accept-encoding` 标头，该标头指定服务器接受的算法。

If the client message is compressed using one of the algorithms from the `grpc-accept-encoding` header and an `UNIMPLEMENTED` error status is returned from the server, the cause of the error won’t be related to compression.

​	如果使用 `grpc-accept-encoding` 标头中的某个算法压缩客户端消息，并且从服务器返回 `UNIMPLEMENTED` 错误状态，则错误原因与压缩无关。

Note that a peer may choose to not disclose all the encodings it supports. However, if it receives a message compressed in an undisclosed but supported encoding, it will include said encoding in the response’s `grpc-accept-encoding` header.

​	请注意，对等方可以选择不公开其支持的所有编码。但是，如果它收到以未公开但受支持的编码压缩的消息，它会将所述编码包含在响应的 `grpc-accept-encoding` 标头中。

For every message a server is requested to compress using an algorithm it knows the client doesn’t support (as indicated by the last `grpc-accept-encoding` header received from the client), it will send the message uncompressed.

​	对于服务器被要求使用客户端不支持的算法（由从客户端收到的最后一个 `grpc-accept-encoding` 标头指示）压缩的每条消息，它都会发送未压缩的消息。

### Specific Disabling of Compression 具体禁用压缩

If the user requests to disable compression, the next message will be sent uncompressed. This is instrumental in preventing [BEAST](https://en.wikipedia.org/wiki/Transport_Layer_Security#BEAST_attack) and [CRIME](https://en.wikipedia.org/wiki/CRIME) attacks. This applies to both the unary and streaming cases.

​	如果用户请求禁用压缩，则下一条消息将以未压缩的形式发送。这有助于防止 BEAST 和 CRIME 攻击。这适用于一元和流式两种情况。

### Language guides and examples 语言指南和示例

| Language 语言 | Example 示例                                                 | Documentation 文档                                           |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| C++           | [C++ Example C++ 示例](https://github.com/grpc/grpc/tree/master/examples/cpp/compression) | [C++ Documentation C++ 文档](https://github.com/grpc/grpc/tree/master/examples/cpp/compression) |
| Go            | [Go Example Go 示例](https://github.com/grpc/grpc-go/tree/master/examples/features/compression) | [Go Documentation Go 文档](https://github.com/grpc/grpc-go/blob/master/Documentation/compression.md) |
| Java          | [Java Example Java 示例](https://github.com/grpc/grpc-java/tree/master/examples/src/main/java/io/grpc/examples/experimental) | [Java Documentation Java 文档](https://grpc.github.io/grpc-java/javadoc/io/grpc/CallOptions.html#withCompression-java.lang.String-) |
| Python        | [Python Example Python 示例](https://github.com/grpc/grpc/tree/master/examples/python/compression) | [Python Documentation Python 文档](https://github.com/grpc/grpc/tree/master/examples/python/compression) |

### Additional Resources 其他资源

- [gRPC Compression gRPC 压缩](https://github.com/grpc/grpc/blob/master/doc/compression.md)
- [gRPC (Core) Compression Cookbook
  gRPC（核心）压缩食谱](https://github.com/grpc/grpc/blob/master/doc/compression_cookbook.md#per-call-settings)
- [gRFC for Python Compression API
  Python 压缩 API 的 gRFC](https://github.com/grpc/proposal/blob/master/L46-python-compression-api.md)
