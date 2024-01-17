+++
title = "FAQ"
date = 2024-01-17T08:51:13+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/what-is-grpc/faq/](https://grpc.io/docs/what-is-grpc/faq/)

# FAQ





Here are some frequently asked questions. Hope you find your answer here :-)

​	这里有一些常见问题。希望您能在这里找到您的答案 :-)

### What is gRPC? 什么是 gRPC？

gRPC is a modern, open source remote procedure call (RPC) framework that can run anywhere. It enables client and server applications to communicate transparently, and makes it easier to build connected systems.

​	gRPC 是一个现代的、开源的远程过程调用 (RPC) 框架，可以在任何地方运行。它使客户端和服务器应用程序能够透明地通信，并使构建连接的系统变得更加容易。

Read the longer [Motivation & Design Principles](https://grpc.io/blog/principles/) post for background on why we created gRPC.

​	阅读更长的动机和设计原则文章，了解我们创建 gRPC 的背景。

### What does gRPC stand for? gRPC 代表什么？

**g**RPC **R**emote **P**rocedure **C**alls, of course!

​	当然，gRPC 远程过程调用！

### Why would I want to use gRPC? 我为什么要使用 gRPC？

The main usage scenarios:

​	主要使用场景：

- Low latency, highly scalable, distributed systems.
  低延迟、高度可扩展的分布式系统。
- Developing mobile clients which are communicating to a cloud server.
  开发与云服务器通信的移动客户端。
- Designing a new protocol that needs to be accurate, efficient and language independent.
  设计需要准确、高效且与语言无关的新协议。
- Layered design to enable extension eg. authentication, load balancing, logging and monitoring etc.
  分层设计以启用扩展，例如身份验证、负载平衡、日志记录和监视等。

### Who’s using this and why? 谁在使用它以及为什么？

gRPC is a [Cloud Native Computing Foundation](https://cncf.io/) (CNCF) project.

​	gRPC 是一个云原生计算基金会 (CNCF) 项目。

Google has been using a lot of the underlying technologies and concepts in gRPC for a long time. The current implementation is being used in several of Google’s cloud products and Google externally facing APIs. It is also being used by [Square](https://corner.squareup.com/2015/02/grpc.html), [Netflix](https://github.com/Netflix/ribbon), [CoreOS](https://blog.gopheracademy.com/advent-2015/etcd-distributed-key-value-store-with-grpc-http2/), [Docker](https://blog.docker.com/2015/12/containerd-daemon-to-control-runc/), [CockroachDB](https://github.com/cockroachdb/cockroach), [Cisco](https://github.com/CiscoDevNet/grpc-getting-started), [Juniper Networks](https://github.com/Juniper/open-nti) and many other organizations and individuals.

​	Google 长期以来一直在使用 gRPC 中的许多底层技术和概念。当前的实现已在 Google 的多个云产品和 Google 外部 API 中使用。它还被 Square、Netflix、CoreOS、Docker、CockroachDB、思科、瞻博网络以及许多其他组织和个人使用。

### Which programming languages are supported? 支持哪些编程语言？

For officially supported languages and platforms, see [Official support](https://grpc.io/docs/#official-support).

​	有关官方支持的语言和平台，请参阅官方支持。

### How do I get started using gRPC? 如何开始使用 gRPC？

You can start with installation of gRPC by following instructions [here](https://grpc.io/docs/quickstart/). Or head over to the [gRPC GitHub org page](https://github.com/grpc), pick the runtime or language you are interested in, and follow the README instructions.

​	您可以按照此处提供的说明开始安装 gRPC。或者前往 gRPC GitHub 组织页面，选择您感兴趣的运行时或语言，然后按照自述文件中的说明进行操作。

### Which license is gRPC under? gRPC 采用哪种许可证？

All implementations are licensed under [Apache 2.0](https://github.com/grpc/grpc/blob/master/LICENSE).

​	所有实现均采用 Apache 2.0 许可证。

### How can I contribute? 如何参与贡献？

[Contributors](https://grpc.io/community/#contribute) are highly welcome and the repositories are hosted on GitHub. We look forward to community feedback, additions and bugs. Both individual contributors and corporate contributors need to sign our CLA. If you have ideas for a project around gRPC, read guidelines and submit [here](https://github.com/grpc/grpc-contrib/blob/master/CONTRIBUTING.md). We have a growing list of projects under the [gRPC Ecosystem](https://github.com/grpc-ecosystem) organization on GitHub.

​	非常欢迎贡献者，代码库托管在 GitHub 上。我们期待社区的反馈、补充和错误报告。个人贡献者和企业贡献者都需要签署我们的 CLA。如果您有关于 gRPC 的项目想法，请阅读指南并在此处提交。我们在 GitHub 上的 gRPC 生态系统组织下有一个不断增长的项目列表。

### Where is the documentation? 文档在哪里？

Check out the [documentation](https://grpc.io/docs/) right here on grpc.io.

​	在此处的 grpc.io 上查看文档。

### What is the road map? 路线图是什么？

The gRPC project has an RFC process, through which new features are designed and approved for implementation. They are tracked in [this repository](https://github.com/grpc/proposal).

​	gRPC 项目有一个 RFC 流程，通过该流程设计新功能并批准实施。它们在此代码库中进行跟踪。

### How long are gRPC releases supported for? gRPC 版本支持多长时间？

The gRPC project does not do LTS releases. Given the rolling release model above, we support the current, latest release and the release prior to that. Support here means bug fixes and security fixes.

​	gRPC 项目不进行 LTS 版本发布。鉴于上述滚动发布模型，我们支持当前的最新版本和之前的版本。此处支持是指错误修复和安全修复。

### What is the gRPC versioning policy? gRPC 版本控制政策是什么？

See the gRPC versioning policy [here](https://github.com/grpc/grpc/blob/master/doc/versioning.md).

​	在此处查看 gRPC 版本控制政策。

### What is the latest gRPC Version? 最新的 gRPC 版本是什么？

The latest release tag is v1.60.0.

​	最新发布版本为 v1.60.0。

### When do gRPC releases happen? gRPC 发布何时发生？

The gRPC project works in a model where the tip of the master branch is stable at all times. The project (across the various runtimes) targets to ship checkpoint releases every 6 weeks on a best effort basis. See the release schedule [here](https://github.com/grpc/grpc/blob/master/doc/grpc_release_schedule.md).

​	gRPC 项目采用一种模型，其中 master 分支的提示始终稳定。该项目（跨各种运行时）的目标是在尽最大努力的基础上每 6 周发布一次检查点版本。在此处查看发布计划。

### How can I report a security vulnerability in gRPC? 如何在 gRPC 中报告安全漏洞？

To report a security vulnerability in gRPC, please follow the process documented [here](https://github.com/grpc/proposal/blob/master/P4-grpc-cve-process.md).

​	要报告 gRPC 中的安全漏洞，请按照此处记录的流程进行操作。

### Can I use it in the browser? 我可以在浏览器中使用它吗？

The [gRPC-Web](https://github.com/grpc/grpc-web) project is Generally Available.

​	gRPC-Web 项目已普遍可用。

### Can I use gRPC with my favorite data format (JSON, Protobuf, Thrift, XML) ? 我可以用 gRPC 和我最喜欢的数据格式（JSON、Protobuf、Thrift、XML）一起使用吗？

Yes. gRPC is designed to be extensible to support multiple content types. The initial release contains support for Protobuf and with external support for other content types such as FlatBuffers and Thrift, at varying levels of maturity.

​	可以。gRPC 被设计为可扩展的，以支持多种内容类型。初始版本包含对 Protobuf 的支持，并以不同成熟度级别外部支持其他内容类型，如 FlatBuffers 和 Thrift。

### Can I use gRPC in a service mesh? 我可以在服务网格中使用 gRPC 吗？

Yes. gRPC applications can be deployed in a service mesh like any other application. gRPC also supports [xDS APIs](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol) which enables deploying gRPC applications in a service mesh without sidecar proxies. The proxyless service mesh features supported in gRPC are listed [here](https://github.com/grpc/grpc/blob/master/doc/grpc_xds_features.md).

​	是的。gRPC 应用程序可以像任何其他应用程序一样部署在服务网格中。gRPC 还支持 xDS API，这使得无需 sidecar 代理即可在服务网格中部署 gRPC 应用程序。此处列出了 gRPC 中支持的无代理服务网格功能。

### How does gRPC help in mobile application development? gRPC 如何帮助移动应用程序开发？

gRPC and Protobuf provide an easy way to precisely define a service and auto generate reliable client libraries for iOS, Android and the servers providing the back end. The clients can take advantage of advanced streaming and connection features which help save bandwidth, do more over fewer TCP connections and save CPU usage and battery life.

​	gRPC 和 Protobuf 提供了一种简单的方法来精确定义服务并自动生成适用于 iOS、Android 和提供后端的服务器的可靠客户端库。客户端可以利用高级流和连接功能，这些功能有助于节省带宽，通过更少的 TCP 连接完成更多工作，并节省 CPU 使用率和电池寿命。

### Why is gRPC better than any binary blob over HTTP/2? 为什么 gRPC 比 HTTP/2 上的任何二进制 blob 都好？

This is largely what gRPC is on the wire. However gRPC is also a set of libraries that will provide higher-level features consistently across platforms that common HTTP libraries typically do not. Examples of such features include:

​	这在很大程度上就是 gRPC 在网络上的样子。但是，gRPC 也是一组库，它将在各个平台上始终如一地提供高级功能，而常见的 HTTP 库通常不提供这些功能。此类功能的示例包括：

- interaction with flow-control at the application layer
  与应用程序层上的流控制交互
- cascading call-cancellation
  级联调用取消
- load balancing & failover
  负载平衡和故障转移

### Why is gRPC better/worse than REST? 为什么 gRPC 比 REST 好/差？

gRPC largely follows HTTP semantics over HTTP/2 but we explicitly allow for full-duplex streaming. We diverge from typical REST conventions as we use static paths for performance reasons during call dispatch as parsing call parameters from paths, query parameters and payload body adds latency and complexity. We have also formalized a set of errors that we believe are more directly applicable to API use cases than the HTTP status codes.

​	gRPC 在 HTTP/2 上在很大程度上遵循 HTTP 语义，但我们明确允许全双工流式传输。我们偏离典型的 REST 约定，因为我们在调用分派期间出于性能原因使用静态路径，因为从路径、查询参数和有效负载正文解析调用参数会增加延迟和复杂性。我们还正式制定了一组错误，我们认为这些错误比 HTTP 状态代码更直接适用于 API 用例。

### How do you pronounce gRPC? gRPC 的发音是什么？

Jee-Arr-Pee-See.
