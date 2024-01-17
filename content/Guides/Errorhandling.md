+++
title = "Error handling"
date = 2024-01-17T08:51:13+08:00
weight = 80
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/guides/error/](https://grpc.io/docs/guides/error/)

# Error handling 错误处理

How gRPC deals with errors, and gRPC error codes.

​	gRPC 如何处理错误以及 gRPC 错误代码。



### Standard error model 标准错误模型

As you’ll have seen in our concepts document and examples, when a gRPC call completes successfully the server returns an `OK` status to the client (depending on the language the `OK` status may or may not be directly used in your code). But what happens if the call isn’t successful?

​	正如您在我们的概念文档和示例中所见，当 gRPC 调用成功完成时，服务器会向客户端返回一个 `OK` 状态（根据语言， `OK` 状态可能直接用于您的代码中，也可能不直接用于您的代码中）。但如果调用不成功会发生什么情况？

If an error occurs, gRPC returns one of its error status codes instead, with an optional string error message that provides further details about what happened. Error information is available to gRPC clients in all supported languages.

​	如果发生错误，gRPC 会返回其错误状态代码之一，并附带一个可选的字符串错误消息，其中提供了有关发生情况的更多详细信息。所有受支持语言中的 gRPC 客户端都可以使用错误信息。

### Richer error model 更丰富的错误模型

The error model described above is the official gRPC error model, is supported by all gRPC client/server libraries, and is independent of the gRPC data format (whether protocol buffers or something else). You may have noticed that it’s quite limited and doesn’t include the ability to communicate error details.

​	上面描述的错误模型是官方 gRPC 错误模型，受所有 gRPC 客户端/服务器库支持，并且与 gRPC 数据格式无关（无论是协议缓冲区还是其他格式）。您可能已经注意到，它非常有限，并且不包括传达错误详细信息的功能。

If you’re using protocol buffers as your data format, however, you may wish to consider using the richer error model developed and used by Google as described [here](https://cloud.google.com/apis/design/errors#error_model). This model enables servers to return and clients to consume additional error details expressed as one or more protobuf messages. It further specifies a [standard set of error message types](https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto) to cover the most common needs (such as invalid parameters, quota violations, and stack traces). The protobuf binary encoding of this extra error information is provided as trailing metadata in the response.

​	但是，如果您使用协议缓冲区作为数据格式，您可能希望考虑使用 Google 开发和使用的更丰富的错误模型，如此处所述。此模型使服务器能够返回，客户端能够使用一个或多个 protobuf 消息表示的附加错误详细信息。它进一步指定了一组标准的错误消息类型，以满足最常见的需求（例如无效参数、配额违规和堆栈跟踪）。此额外错误信息的 protobuf 二进制编码作为响应中的尾部元数据提供。

This richer error model is already supported in the C++, Go, Java, Python, and Ruby libraries, and at least the grpc-web and Node.js libraries have open issues requesting it. Other language libraries may add support in the future if there’s demand, so check their github repos if interested. Note however that the grpc-core library written in C will not likely ever support it since it is purposely data format agnostic.

​	这种更丰富的错误模型已经在 C++、Go、Java、Python 和 Ruby 库中得到支持，并且至少 grpc-web 和 Node.js 库有开放的问题要求它。如果需求存在，其他语言库可能会在未来添加支持，因此如果有兴趣，请查看它们的 github 存储库。但请注意，用 C 编写的 grpc-core 库可能永远不会支持它，因为它故意与数据格式无关。

You could use a similar approach (put error details in trailing response metadata) if you’re not using protocol buffers, but you’d likely need to find or develop library support for accessing this data in order to make practical use of it in your APIs.

​	如果您不使用协议缓冲区，则可以使用类似的方法（将错误详细信息放入尾随响应元数据中），但您可能需要查找或开发库支持来访问此数据，以便在 API 中实际使用它。

There are important considerations to be aware of when deciding whether to use such an extended error model, however, including:

​	但是，在决定是否使用这种扩展的错误模型时，需要注意一些重要的考虑因素，包括：

- Library implementations of the extended error model may not be consistent across languages in terms of requirements for and expectations of the error details payload
  扩展错误模型的库实现可能在不同语言中不一致，这取决于对错误详细信息有效负载的要求和期望
- Existing proxies, loggers, and other standard HTTP request processors don’t have visibility into the error details and thus wouldn’t be able to leverage them for monitoring or other purposes
  现有的代理、记录器和其他标准 HTTP 请求处理器无法查看错误详细信息，因此无法利用它们进行监视或其他目的
- Additional error detail in the trailers interferes with head-of-line blocking, and will decrease HTTP/2 header compression efficiency due to more frequent cache misses
  预告片中的其他错误详细信息会干扰行首阻塞，并且由于更频繁的缓存未命中，这会降低 HTTP/2 标头压缩效率
- Larger error detail payloads may run into protocol limits (like max headers size), effectively losing the original error
  较大的错误详细信息有效负载可能会遇到协议限制（例如最大标头大小），从而有效地丢失原始错误

### Error status codes 错误状态代码

Errors are raised by gRPC under various circumstances, from network failures to unauthenticated connections, each of which is associated with a particular status code. The following error status codes are supported in all gRPC languages.

​	gRPC 在各种情况下都会引发错误，从网络故障到未经身份验证的连接，每种情况都与特定的状态代码相关联。以下错误状态代码在所有 gRPC 语言中均受支持。

#### General errors 常规错误

| Case 情况                                                    | Status code 状态代码            |
| ------------------------------------------------------------ | ------------------------------- |
| Client application cancelled the request 客户端应用程序取消了请求 | `GRPC_STATUS_CANCELLED`         |
| Deadline expired before server returned status 在服务器返回状态之前超时 | `GRPC_STATUS_DEADLINE_EXCEEDED` |
| Method not found on server 服务器上未找到方法                | `GRPC_STATUS_UNIMPLEMENTED`     |
| Server shutting down 服务器正在关闭                          | `GRPC_STATUS_UNAVAILABLE`       |
| Server threw an exception (or did something other than returning a status code to terminate the RPC) 服务器抛出异常（或执行除向 RPC 返回状态代码之外的其他操作以终止 RPC） | `GRPC_STATUS_UNKNOWN`           |

#### Network failures 网络故障

| Case 情况                                                    | Status code 状态码              |
| ------------------------------------------------------------ | ------------------------------- |
| No data transmitted before deadline expires. Also applies to cases where some data is transmitted and no other failures are detected before the deadline expires 在截止日期到期前未传输任何数据。也适用于在截止日期到期前已传输部分数据且未检测到其他故障的情况 | `GRPC_STATUS_DEADLINE_EXCEEDED` |
| Some data transmitted (for example, the request metadata has been written to the TCP connection) before the connection breaks 在连接中断前已传输部分数据（例如，请求元数据已写入 TCP 连接） | `GRPC_STATUS_UNAVAILABLE`       |

#### Protocol errors 协议错误

| Case 情况                                                    | Status code 状态码               |
| ------------------------------------------------------------ | -------------------------------- |
| Could not decompress but compression algorithm supported 无法解压缩，但支持压缩算法 | `GRPC_STATUS_INTERNAL`           |
| Compression mechanism used by client not supported by the server 服务器不支持客户端使用的压缩机制 | `GRPC_STATUS_UNIMPLEMENTED`      |
| Flow-control resource limits reached 达到流控制资源限制      | `GRPC_STATUS_RESOURCE_EXHAUSTED` |
| Flow-control protocol violation 流控制协议违规               | `GRPC_STATUS_INTERNAL`           |
| Error parsing returned status 错误解析返回状态               | `GRPC_STATUS_UNKNOWN`            |
| Unauthenticated: credentials failed to get metadata 未经身份验证：凭据无法获取元数据 | `GRPC_STATUS_UNAUTHENTICATED`    |
| Invalid host set in authority metadata 在权限元数据中设置了无效主机 | `GRPC_STATUS_UNAUTHENTICATED`    |
| Error parsing response protocol buffer 错误解析响应协议缓冲区 | `GRPC_STATUS_INTERNAL`           |
| Error parsing request protocol buffer 错误解析请求协议缓冲区 | `GRPC_STATUS_INTERNAL`           |

### Language Support 语言支持

Examples code is avilable for multiple languages on how to deal with standard errors as well as with the richer error details.

​	多个语言的示例代码可用于处理标准错误以及更丰富的错误详细信息。

| Language 语言 | Example 示例                                                 |
| ------------- | ------------------------------------------------------------ |
| C++           | [C++ error handling example C++ 错误处理示例](https://github.com/grpc/grpc/tree/master/examples/cpp/error_handling) |
|               | [C++ error details example C++ 错误详细信息示例](https://github.com/grpc/grpc/tree/master/examples/cpp/error_details) |
| Go            | [Go error handling example Go 错误处理示例](https://github.com/grpc/grpc-go/tree/master/examples/features/error_handling) |
|               | [Go error details example Go 错误详情示例](https://github.com/grpc/grpc-go/tree/master/examples/features/error_details) |
| Java          | [Java error handling example Java 错误处理示例](https://github.com/grpc/grpc-java/tree/master/examples/src/main/java/io/grpc/examples/errorhandling) |
|               | [Java error details example Java 错误详情示例](https://github.com/grpc/grpc-java/tree/master/examples/src/main/java/io/grpc/examples/errordetails) |
| Node          | [Node error handling example Node 错误处理示例](https://github.com/grpc/grpc-node/tree/master/examples/error_handling) |
| Python        | [Python error details example Python 错误详情示例](https://github.com/grpc/grpc/tree/master/examples/python/errors) |

The [grpc-errors](https://github.com/avinassh/grpc-errors) repo also contains additional error handling examples.

​	grpc-errors 代码库还包含其他错误处理示例。
