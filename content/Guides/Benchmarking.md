+++
title = "Benchmarking"
date = 2024-01-17T08:51:13+08:00
weight = 10
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/guides/benchmarking/](https://grpc.io/docs/guides/benchmarking/)

# Benchmarking 基准测试

gRPC is designed to support high-performance open-source RPCs in many languages. This page describes performance benchmarking tools, scenarios considered by tests, and the testing infrastructure.

​	gRPC 旨在支持多种语言的高性能开源 RPC。此页面介绍性能基准测试工具、测试考虑的情景以及测试基础架构。



### Overview 概述

gRPC is designed for both high-performance and high-productivity design of distributed applications. Continuous performance benchmarking is a critical part of the gRPC development workflow. Multi-language performance tests run every few hours against the master branch, and these numbers are reported to a dashboard for visualization.

​	gRPC 旨在实现高性能和高生产力的分布式应用程序设计。持续性能基准测试是 gRPC 开发工作流的关键部分。多语言性能测试每隔几小时针对主分支运行一次，并将这些数字报告给仪表板以进行可视化。

- [Multi-language performance dashboard @master (latest dev version)
  多语言性能仪表板 @master（最新开发版本）](https://grafana-dot-grpc-testing.appspot.com/)
- [Legacy dashboard (same data as above)
  传统信息中心（与上述数据相同）](https://performance-dot-grpc-testing.appspot.com/explore?dashboard=5180705743044608)

### Performance testing design 性能测试设计

Each language implements a performance testing worker that implements a gRPC [WorkerService](https://github.com/grpc/grpc/blob/master/src/proto/grpc/testing/worker_service.proto). This service directs the worker to act as either a client or a server for the actual benchmark test, represented as [BenchmarkService](https://github.com/grpc/grpc/blob/master/src/proto/grpc/testing/benchmark_service.proto). That service has two methods:

​	每种语言都实现了一个性能测试工作程序，该工作程序实现了一个 gRPC WorkerService。此服务指示工作程序充当实际基准测试的客户端或服务器，表示为 BenchmarkService。该服务具有两种方法：

- UnaryCall – a unary RPC of a simple request that specifies the number of bytes to return in the response.
  UnaryCall - 单个 RPC，用于指定响应中要返回的字节数的简单请求。
- StreamingCall – a streaming RPC that allows repeated ping-pongs of request and response messages akin to the UnaryCall.
  StreamingCall - 流式 RPC，允许对请求和响应消息进行类似于 UnaryCall 的重复 ping-pong 操作。



![gRPC performance testing worker diagram](./Benchmarking_img/testing_framework.png)



These workers are controlled by a [driver](https://github.com/grpc/grpc/blob/master/test/cpp/qps/qps_json_driver.cc) that takes as input a scenario description (in JSON format) and an environment variable specifying the host:port of each worker process.

​	这些工作程序由一个驱动程序控制，该驱动程序将场景描述（JSON 格式）和指定每个工作程序进程的主机:端口的环境变量作为输入。

### Languages under test 受测语言

The following languages have continuous performance testing as both clients and servers at master:

​	以下语言在 master 上同时作为客户端和服务器进行持续性能测试：

- C++
- Java
- Go
- C#
- Node.js
- Python
- Ruby

In addition to running as both the client-side and server-side of performance tests, all languages are tested as clients against a C++ server, and as servers against a C++ client. This test aims to provide the current upper bound of performance for a given language’s client or server implementation without testing the other side.

​	除了作为性能测试的客户端和服务器端运行外，所有语言都作为客户端针对 C++ 服务器进行测试，并作为服务器针对 C++ 客户端进行测试。此测试旨在提供给定语言的客户端或服务器实现的当前性能上限，而无需测试另一方。

Although PHP or mobile environments do not support a gRPC server (which is needed for our performance tests), their client-side performance can be benchmarked using a proxy WorkerService written in another language. This code is implemented for PHP but is not yet in continuous testing mode.

​	虽然 PHP 或移动环境不支持 gRPC 服务器（这是我们的性能测试所必需的），但可以使用另一种语言编写的代理 WorkerService 对其客户端性能进行基准测试。此代码已针对 PHP 实现，但尚未处于持续测试模式。

### Scenarios under test 测试中的场景

There are several important scenarios under test and displayed in the dashboards above, including the following:

​	在上面的信息中心中测试并显示了几个重要的场景，包括以下内容：

- Contentionless latency – the median and tail response latencies seen with only 1 client sending a single message at a time using StreamingCall.
  无竞争延迟 - 仅使用 1 个客户端通过 StreamingCall 一次发送一条消息时看到的中间和尾部响应延迟。
- QPS – the messages/second rate when there are 2 clients and a total of 64 channels, each of which has 100 outstanding messages at a time sent using StreamingCall.
  QPS - 当有 2 个客户端和总共 64 个通道时每秒的消息数，每个通道一次有 100 条未完成的消息通过 StreamingCall 发送。
- Scalability (for selected languages) – the number of messages/second per server core.
  可扩展性（针对选定语言） - 每秒每服务器内核的消息数。

Most performance testing is using secure communication and protobufs. Some C++ tests additionally use insecure communication and the generic (non-protobuf) API to display peak performance. Additional scenarios may be added in the future.

​	大多数性能测试都使用安全通信和 Protocol Buffers。一些 C++ 测试还使用不安全的通信和通用（非 Protocol Buffers）API 来显示峰值性能。未来可能会添加其他场景。

### Testing infrastructure 测试基础架构

All performance benchmarks are run in our dedicated GKE cluster, where each benchmark worker (a client or a server) gets scheduled to different GKE node (and each GKE node is a separate GCE VM) in one of our worker pools. The source code for the benchmarking framework we use is publicly available in the [test-infra github repository](https://github.com/grpc/test-infra).

​	所有性能基准测试都在我们专用的 GKE 集群中运行，其中每个基准测试工作进程（客户端或服务器）都会被调度到我们某个工作程序池中的不同 GKE 节点（每个 GKE 节点都是一个单独的 GCE VM）。我们使用的基准测试框架的源代码在 test-infra github 存储库中公开提供。

Most test instances are 8-core systems, and these are used for both latency and QPS measurement. For C++ and Java, we additionally support QPS testing on 32-core systems. All QPS tests use 2 identical client machines for each server, to make sure that QPS measurement is not client-limited.

​	大多数测试实例都是 8 核系统，这些系统用于延迟和 QPS 测量。对于 C++ 和 Java，我们还支持在 32 核系统上进行 QPS 测试。所有 QPS 测试都为每个服务器使用 2 台相同的客户端计算机，以确保 QPS 测量不受客户端限制。
