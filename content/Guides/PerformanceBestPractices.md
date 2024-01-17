+++
title = "Performance Best Practices"
date = 2024-01-17T08:51:13+08:00
weight = 130
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/guides/performance/](https://grpc.io/docs/guides/performance/)

# Performance Best Practices 性能最佳实践

A user guide of both general and language-specific best practices to improve performance.

​	通用和特定于语言的最佳实践用户指南，以提高性能。



### General 一般

- Always **re-use stubs and channels** when possible.

  ​	尽可能始终重复使用存根和通道。

- **Use keepalive pings** to keep HTTP/2 connections alive during periods of inactivity to allow initial RPCs to be made quickly without a delay (i.e. C++ channel arg GRPC_ARG_KEEPALIVE_TIME_MS).

  ​	在不活动期间使用保活 ping 来保持 HTTP/2 连接处于活动状态，以便在没有延迟的情况下快速进行初始 RPC（即 C++ 通道参数 GRPC_ARG_KEEPALIVE_TIME_MS）。

- **Use streaming RPCs** when handling a long-lived logical flow of data from the client-to-server, server-to-client, or in both directions. Streams can avoid continuous RPC initiation, which includes connection load balancing at the client-side, starting a new HTTP/2 request at the transport layer, and invoking a user-defined method handler on the server side.

  ​	在处理从客户端到服务器、从服务器到客户端或双向的长期逻辑数据流时，使用流式 RPC。流可以避免连续的 RPC 初始化，其中包括客户端的连接负载均衡、在传输层启动新的 HTTP/2 请求以及在服务器端调用用户定义的方法处理程序。

  Streams, however, cannot be load balanced once they have started and can be hard to debug for stream failures. They also might increase performance at a small scale but can reduce scalability due to load balancing and complexity, so they should only be used when they provide substantial performance or simplicity benefit to application logic. Use streams to optimize the application, not gRPC.

  ​	但是，流一旦启动就无法进行负载均衡，并且在流发生故障时可能难以进行调试。它们还可能在小规模时提高性能，但由于负载均衡和复杂性而降低可扩展性，因此仅应在为应用程序逻辑提供实质性性能或简单性优势时才使用它们。使用流来优化应用程序，而不是 gRPC。

  ***Side note:*** *This does not apply to Python (see Python section for details).*

  ​	旁注：这不适用于 Python（有关详细信息，请参阅 Python 部分）。

- *(Special topic)* Each gRPC channel uses 0 or more HTTP/2 connections and each connection usually has a limit on the number of concurrent streams. When the number of active RPCs on the connection reaches this limit, additional RPCs are queued in the client and must wait for active RPCs to finish before they are sent. Applications with high load or long-lived streaming RPCs might see performance issues because of this queueing. There are two possible solutions:

  ​	（特殊主题）每个 gRPC 通道使用 0 个或多个 HTTP/2 连接，每个连接通常对并发流的数量有限制。当连接上的活动 RPC 数量达到此限制时，其他 RPC 会在客户端排队，并且必须等到活动 RPC 完成后才能发送。具有高负载或长寿命流式 RPC 的应用程序可能会因这种排队而出现性能问题。有两种可能的解决方案：

  1. **Create a separate channel for each area of high load** in the application.

     ​	为应用程序中的每个高负载区域创建一个单独的通道。

  2. **Use a pool of gRPC channels** to distribute RPCs over multiple connections (channels must have different channel args to prevent re-use so define a use-specific channel arg such as channel number).

     ​	使用 gRPC 通道池将 RPC 分发到多个连接（通道必须具有不同的通道参数以防止重复使用，因此定义一个特定于用途的通道参数，例如通道号）。

  ***Side note:*** *The gRPC team has plans to add a feature to fix these performance issues (see [grpc/grpc#21386](https://github.com/grpc/grpc/issues/21386) for more info), so any solution involving creating multiple channels is a temporary workaround that should eventually not be needed.*

  ​	旁注：gRPC 团队计划添加一项功能来修复这些性能问题（有关更多信息，请参阅 grpc/grpc#21386），因此涉及创建多个通道的任何解决方案都是一种临时解决方法，最终应该不需要。

### C++

- **Do not use Sync API for performance sensitive servers.** If performance and/or resource consumption are not concerns, use the Sync API as it is the simplest to implement for low-QPS services.

  ​	不要对性能敏感的服务器使用同步 API。如果性能和/或资源消耗不是问题，请使用同步 API，因为它是为低 QPS 服务实现的最简单的 API。

- **Favor callback API over other APIs for most RPCs**, given that the application can avoid all blocking operations or blocking operations can be moved to a separate thread. The callback API is easier to use than the completion-queue async API but is currently slower for truly high-QPS workloads.

  ​	鉴于应用程序可以避免所有阻塞操作或将阻塞操作移至单独的线程，因此对于大多数 RPC，优先使用回调 API 而非其他 API。回调 API 比完成队列异步 API 更易于使用，但对于真正的高 QPS 工作负载而言，目前速度较慢。

- If having to use the async completion-queue API, the **best scalability trade-off is having `numcpu`’s threads.** The ideal number of completion queues in relation to the number of threads can change over time (as gRPC C++ evolves), but as of gRPC 1.41 (Sept 2021), using 2 threads per completion queue seems to give the best performance.

  ​	如果必须使用异步完成队列 API，则最佳的可伸缩性权衡是拥有 `numcpu` 的线程。完成队列的理想数量与线程数量之间的关系可能会随着时间而改变（随着 gRPC C++ 的发展），但截至 gRPC 1.41（2021 年 9 月），每个完成队列使用 2 个线程似乎可以提供最佳性能。

- For the async completion-queue API, make sure to **register enough server requests for the desired level of concurrency** to avoid the server continuously getting stuck in a slow path that results in essentially serial request processing.

  ​	对于异步完成队列 API，请务必注册足够的服务器请求以达到所需的并发级别，以避免服务器持续陷入导致基本串行请求处理的慢速路径。

- *(Special topic)* **Enable write batching in streams** if message k + 1 does not rely on responses from message k by passing a WriteOptions argument to Write with buffer_hint set:

  ​	（特殊主题）如果消息 k + 1 不依赖于消息 k 的响应，则通过将 WriteOptions 参数传递给 Write 并设置 buffer_hint 来启用流中的写入批处理：
  `stream_writer->Write(message, WriteOptions().set_buffer_hint());`

- *(Special topic)* [gRPC::GenericStub](https://grpc.github.io/grpc/cpp/grpcpp_2generic_2generic__stub_8h.html) can be useful in certain cases when there is high contention / CPU time spent on proto serialization. This class allows the application to directly send **raw gRPC::ByteBuffer as data** rather than serializing from some proto. This can also be helpful if the same data is being sent multiple times, with one explicit proto-to-ByteBuffer serialization followed by multiple ByteBuffer sends.

  ​	（特殊主题）在某些情况下，当在 proto 序列化上花费大量争用/CPU 时间时，gRPC::GenericStub 会很有用。此类允许应用程序直接发送原始 gRPC::ByteBuffer 作为数据，而不是从某个 proto 序列化。如果多次发送相同数据，这也会很有用，只需进行一次显式 proto 到 ByteBuffer 的序列化，然后发送多个 ByteBuffer。

### Java

- **Use non-blocking stubs** to parallelize RPCs.

  ​	使用非阻塞存根来并行化 RPC。

- **Provide a custom executor that limits the number of threads, based on your workload** (cached (default), fixed, forkjoin, etc).

  ​	提供一个自定义执行器，根据您的工作负载限制线程数（缓存（默认）、固定、forkjoin 等）。

### Python

- Streaming RPCs create extra threads for receiving and possibly sending the messages, which makes **streaming RPCs much slower than unary RPCs** in gRPC Python, unlike the other languages supported by gRPC.

  ​	流式 RPC 为接收和可能发送消息创建额外的线程，这使得流式 RPC 比 gRPC Python 中的单一 RPC 慢得多，这与 gRPC 支持的其他语言不同。

- **Using [asyncio](https://grpc.github.io/grpc/python/grpc_asyncio.html)** could improve performance.

  ​	使用 asyncio 可以提高性能。

- Using the future API in the sync stack results in the creation of an extra thread. **Avoid the future API** if possible.

  ​	在同步堆栈中使用 future API 会导致创建额外的线程。如果可能，请避免使用 future API。

- *(Experimental)* An experimental **single-threaded unary-stream implementation** is available via the [SingleThreadedUnaryStream channel option](https://github.com/grpc/grpc/blob/master/src/python/grpcio/grpc/experimental/__init__.py#L38), which can save up to 7% latency per message.

  ​	（实验性）可以通过 SingleThreadedUnaryStream 通道选项使用实验性的单线程单一流实现，这可以为每条消息节省高达 7% 的延迟。
