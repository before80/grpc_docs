+++
title = "ALTS"
date = 2024-01-17T08:51:13+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/python/alts/](https://grpc.io/docs/languages/python/alts/)

# ALTS authentication ALTS 身份验证

An overview of gRPC authentication in Python using Application Layer Transport Security (ALTS).

​	使用应用程序层传输安全 (ALTS) 在 Python 中进行 gRPC 身份验证的概述。



### Overview 概述

Application Layer Transport Security (ALTS) is a mutual authentication and transport encryption system developed by Google. It is used for securing RPC communications within Google’s infrastructure. ALTS is similar to mutual TLS but has been designed and optimized to meet the needs of Google’s production environments. For more information, take a look at the [ALTS whitepaper](https://cloud.google.com/security/encryption-in-transit/application-layer-transport-security).

​	应用程序层传输安全 (ALTS) 是由 Google 开发的相互身份验证和传输加密系统。它用于保护 Google 基础架构内的 RPC 通信。ALTS 与相互 TLS 类似，但经过设计和优化，可满足 Google 生产环境的需求。有关更多信息，请参阅 ALTS 白皮书。

ALTS in gRPC has the following features:

​	gRPC 中的 ALTS 具有以下功能：

- Create gRPC servers & clients with ALTS as the transport security protocol.
  使用 ALTS 作为传输安全协议创建 gRPC 服务器和客户端。
- ALTS connections are end-to-end protected with privacy and integrity.
  ALTS 连接通过隐私和完整性得到端到端保护。
- Applications can access peer information such as the peer service account.
  应用程序可以访问对等信息，例如对等服务帐号。
- Client authorization and server authorization support.
  支持客户端授权和服务器授权。
- Minimal code changes to enable ALTS.
  启用 ALTS 的代码更改最少。

gRPC users can configure their applications to use ALTS as a transport security protocol with few lines of code.

​	gRPC 用户只需几行代码即可配置其应用程序，以使用 ALTS 作为传输安全协议。

Note that ALTS is fully functional if the application runs on [Compute Engine](https://cloud.google.com/compute) or [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine).

​	请注意，如果应用程序在 Compute Engine 或 Google Kubernetes Engine (GKE) 上运行，则 ALTS 完全可用。

### gRPC Client with ALTS Transport Security Protocol 具有 ALTS 传输安全协议的 gRPC 客户端

gRPC clients can use ALTS credentials to connect to servers, as illustrated in the following code excerpt:

​	gRPC 客户端可以使用 ALTS 凭据连接到服务器，如下面的代码摘录所示：

```python
import grpc

channel_creds = grpc.alts_channel_credentials()
channel = grpc.secure_channel(address, channel_creds)
```

### gRPC Server with ALTS Transport Security Protocol 具有 ALTS 传输安全协议的 gRPC 服务器

gRPC servers can use ALTS credentials to allow clients to connect to them, as illustrated next:

​	gRPC 服务器可以使用 ALTS 凭据允许客户端连接到它们，如下所示：

```python
import grpc

server = grpc.server(futures.ThreadPoolExecutor())
server_creds = grpc.alts_server_credentials()
server.add_secure_port(server_address, server_creds)
```
