+++
title = "FAQ"
date = 2024-11-19T10:19:42+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/what-is-grpc/faq/](https://grpc.io/docs/what-is-grpc/faq/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# FAQ





Here are some frequently asked questions. Hope you find your answer here :-)

​	以下是一些常见问题，希望您能在这里找到答案 :-)

### What is gRPC?

gRPC is a modern, open source remote procedure call (RPC) framework that can run anywhere. It enables client and server applications to communicate transparently, and makes it easier to build connected systems.

​	gRPC 是一个现代的、开源的远程过程调用（RPC）框架，可以在任何地方运行。它使客户端和服务器应用程序能够透明地通信，并简化了构建互联系统的过程。

Read the longer [Motivation & Design Principles]({{< ref "/blog/gRPCMotivationandDesignPrinciples" >}}) post for background on why we created gRPC.

​	阅读更详细的 [动机与设计原则]({{< ref "/blog/gRPCMotivationandDesignPrinciples" >}}) 了解创建 gRPC 的背景。

### gRPC 是什么意思？ What does gRPC stand for?

**g**RPC **R**emote **P**rocedure **C**alls, of course!

​	当然是 **g**RPC **R**emote **P**rocedure **C**alls（远程过程调用）！

### 为什么要使用 gRPC？ Why would I want to use gRPC?

The main usage scenarios:

​	主要的使用场景包括：

- Low latency, highly scalable, distributed systems.

  - 低延迟、高度可扩展的分布式系统。

- Developing mobile clients which are communicating to a cloud server.

  - 开发与云服务器通信的移动客户端。

- Designing a new protocol that needs to be accurate, efficient and language independent.

  - 设计需要精确、高效且语言无关的新协议。

- Layered design to enable extension eg. authentication, load balancing, logging and monitoring etc.

  - 分层设计以支持扩展，例如认证、负载均衡、日志记录和监控等。

  

### Who’s using this and why?

gRPC is a [Cloud Native Computing Foundation](https://cncf.io/) (CNCF) project.

​	gRPC 是 [云原生计算基金会](https://cncf.io/)（CNCF）的一个项目。

Google has been using a lot of the underlying technologies and concepts in gRPC for a long time. The current implementation is being used in several of Google’s cloud products and Google externally facing APIs. It is also being used by [Square](https://corner.squareup.com/2015/02/grpc.html), [Netflix](https://github.com/Netflix/ribbon), [CoreOS](https://blog.gopheracademy.com/advent-2015/etcd-distributed-key-value-store-with-grpc-http2/), [Docker](https://blog.docker.com/2015/12/containerd-daemon-to-control-runc/), [CockroachDB](https://github.com/cockroachdb/cockroach), [Cisco](https://github.com/CiscoDevNet/grpc-getting-started), [Juniper Networks](https://github.com/Juniper/open-nti) and many other organizations and individuals.

​	Google 长期以来一直在使用 gRPC 的许多底层技术和概念。当前的实现被用于多个 Google 的云产品和对外 API。其他用户还包括 [Square](https://corner.squareup.com/2015/02/grpc.html)、[Netflix](https://github.com/Netflix/ribbon)、[CoreOS](https://blog.gopheracademy.com/advent-2015/etcd-distributed-key-value-store-with-grpc-http2/)、[Docker](https://blog.docker.com/2015/12/containerd-daemon-to-control-runc/)、[CockroachDB](https://github.com/cockroachdb/cockroach)、[Cisco](https://github.com/CiscoDevNet/grpc-getting-started)、[Juniper Networks](https://github.com/Juniper/open-nti) 等众多组织和个人。

### 支持哪些编程语言？ Which programming languages are supported?

For officially supported languages and platforms, see [Official support](https://grpc.io/docs/#official-support).

​	有关官方支持的语言和平台，请参见 [官方支持](https://grpc.io/docs/#official-support)。

### 如何开始使用 gRPC？How do I get started using gRPC?

You can start with installation of gRPC by following instructions [here](https://grpc.io/docs/quickstart/). Or head over to the [gRPC GitHub org page](https://github.com/grpc), pick the runtime or language you are interested in, and follow the README instructions.

​	您可以按照 [此处](https://grpc.io/docs/quickstart/) 的说明安装 gRPC。或者访问 [gRPC 的 GitHub 组织页面](https://github.com/grpc)，选择您感兴趣的运行时或语言，并按照 README 中的说明操作。

### gRPC 使用的是什么许可证？ Which license is gRPC under?

All implementations are licensed under [Apache 2.0](https://github.com/grpc/grpc/blob/master/LICENSE).

​	所有实现都采用 [Apache 2.0](https://github.com/grpc/grpc/blob/master/LICENSE) 许可证。

### 我如何贡献代码？How can I contribute?

[Contributors](https://grpc.io/community/#contribute) are highly welcome and the repositories are hosted on GitHub. We look forward to community feedback, additions and bugs. Both individual contributors and corporate contributors need to sign our CLA. If you have ideas for a project around gRPC, read guidelines and submit [here](https://github.com/grpc/grpc-contrib/blob/master/CONTRIBUTING.md). We have a growing list of projects under the [gRPC Ecosystem](https://github.com/grpc-ecosystem) organization on GitHub.

​	[贡献者](https://grpc.io/community/#contribute) 非常欢迎，相关代码库托管在 GitHub 上。我们期待社区的反馈、补充和错误修复。个人贡献者和企业贡献者均需签署 CLA。如果您有关于 gRPC 的项目想法，请阅读指南并在 [此处](https://github.com/grpc/grpc-contrib/blob/master/CONTRIBUTING.md) 提交。GitHub 上的 [gRPC Ecosystem](https://github.com/grpc-ecosystem) 组织下有一系列不断增长的项目。

### 文档在哪里？ Where is the documentation?

Check out the [documentation](https://grpc.io/docs/) right here on grpc.io.

​	请查看 [gRPC 文档](https://grpc.io/docs/)，尽在 grpc.io。

### gRPC 的路线图是什么？What is the road map?

The gRPC project has an RFC process, through which new features are designed and approved for implementation. They are tracked in [this repository](https://github.com/grpc/proposal).

​	gRPC 项目有一个 RFC 过程，通过该过程设计并批准新功能的实现。这些内容在 [此代码库](https://github.com/grpc/proposal) 中进行跟踪。

### gRPC 的发布支持时间是多长？How long are gRPC releases supported for?

The gRPC project does not do LTS releases. Given the rolling release model above, we support the current, latest release and the release prior to that. Support here means bug fixes and security fixes.

​	gRPC 项目不发布 LTS（长期支持）版本。按照滚动发布模型，我们支持当前最新的发布版本和前一个发布版本。支持包括错误修复和安全修复。

### gRPC 的版本管理策略是什么？What is the gRPC versioning policy?

See the gRPC versioning policy [here](https://github.com/grpc/grpc/blob/master/doc/versioning.md).

​	请参见 [gRPC 版本管理策略](https://github.com/grpc/grpc/blob/master/doc/versioning.md)。

### 最新的 gRPC 版本是多少？ What is the latest gRPC Version?

The latest release tag is v1.66.0.

​	最新的版本标签是 v1.66.0。

### gRPC 发布的时间是？When do gRPC releases happen?

The gRPC project works in a model where the tip of the master branch is stable at all times. The project (across the various runtimes) targets to ship checkpoint releases every 6 weeks on a best effort basis. See the release schedule [here](https://github.com/grpc/grpc/blob/master/doc/grpc_release_schedule.md).

​	gRPC 项目采用一种模型，其中主分支的最新代码始终保持稳定。项目（涵盖各种运行时）目标是在每 6 周的最佳努力基础上发布检查点版本。请参阅 [发布计划](https://github.com/grpc/grpc/blob/master/doc/grpc_release_schedule.md)。

### 如何报告 gRPC 中的安全漏洞？ How can I report a security vulnerability in gRPC?

To report a security vulnerability in gRPC, please follow the process documented [here](https://github.com/grpc/proposal/blob/master/P4-grpc-cve-process.md).

​	要报告 gRPC 中的安全漏洞，请按照 [此处](https://github.com/grpc/proposal/blob/master/P4-grpc-cve-process.md) 记录的流程操作。

### 我可以在浏览器中使用 gRPC 吗？Can I use it in the browser?

The [gRPC-Web](https://github.com/grpc/grpc-web) project is Generally Available.

​	[gRPC-Web](https://github.com/grpc/grpc-web) 项目已全面可用。

### 我可以使用我喜欢的数据格式（JSON、Protobuf、Thrift、XML）与 gRPC 配合吗？ Can I use gRPC with my favorite data format (JSON, Protobuf, Thrift, XML) ?

Yes. gRPC is designed to be extensible to support multiple content types. The initial release contains support for Protobuf and with external support for other content types such as FlatBuffers and Thrift, at varying levels of maturity.

​	可以。gRPC 设计为可扩展以支持多种内容类型。初始版本支持 Protobuf，并且支持其他内容类型（例如 FlatBuffers 和 Thrift）的外部支持，成熟度有所不同。

### 我可以在服务网格中使用 gRPC 吗？Can I use gRPC in a service mesh?

Yes. gRPC applications can be deployed in a service mesh like any other application. gRPC also supports [xDS APIs](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol) which enables deploying gRPC applications in a service mesh without sidecar proxies. The proxyless service mesh features supported in gRPC are listed [here](https://github.com/grpc/grpc/blob/master/doc/grpc_xds_features.md).

​	可以。gRPC 应用程序可以像其他应用程序一样部署到服务网格中。gRPC 还支持 [xDS APIs](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol)，允许在没有 Sidecar 代理的情况下将 gRPC 应用程序部署到服务网格中。gRPC 支持的无代理服务网格功能列表见 [此处](https://github.com/grpc/grpc/blob/master/doc/grpc_xds_features.md)。

### gRPC 如何帮助移动应用程序开发？How does gRPC help in mobile application development?

gRPC and Protobuf provide an easy way to precisely define a service and auto generate reliable client libraries for iOS, Android and the servers providing the back end. The clients can take advantage of advanced streaming and connection features which help save bandwidth, do more over fewer TCP connections and save CPU usage and battery life.

​	gRPC 和 Protobuf 提供了一种简便的方式来精确定义服务，并为 iOS、Android 和提供后端服务的服务器自动生成可靠的客户端库。客户端可以利用高级流和连接功能，帮助节省带宽、减少 TCP 连接数量并降低 CPU 使用率和电池消耗。

### 为什么 gRPC 优于任何基于 HTTP/2 的二进制 blob？Why is gRPC better than any binary blob over HTTP/2?

This is largely what gRPC is on the wire. However gRPC is also a set of libraries that will provide higher-level features consistently across platforms that common HTTP libraries typically do not. Examples of such features include:

​	gRPC 在传输层本质上是这样的。但 gRPC 还是一组库，提供了跨平台一致的高级功能，而普通 HTTP 库通常不具备这些功能。例如：

- interaction with flow-control at the application layer
  - 应用层的流量控制交互

- cascading call-cancellation
  - 级联调用取消

- load balancing & failover
  - 负载均衡和故障转移


### 为什么 gRPC 比 REST 更好或更差？ Why is gRPC better/worse than REST?

gRPC largely follows HTTP semantics over HTTP/2 but we explicitly allow for full-duplex streaming. We diverge from typical REST conventions as we use static paths for performance reasons during call dispatch as parsing call parameters from paths, query parameters and payload body adds latency and complexity. We have also formalized a set of errors that we believe are more directly applicable to API use cases than the HTTP status codes.

​	gRPC 在 HTTP/2 上大体遵循 HTTP 语义，但明确允许全双工流。我们在调用分派期间使用静态路径以提高性能，与典型的 REST 约定不同，因为从路径、查询参数和负载正文解析调用参数会增加延迟和复杂性。我们还制定了一组正式错误，与 HTTP 状态码相比，我们认为这些错误更适用于 API 用例。

### gRPC 怎么发音？How do you pronounce gRPC?

Jee-Arr-Pee-See.
