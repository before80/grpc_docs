+++
title = "ALTS 认证"
date = 2024-11-19T10:19:42+08:00
weight = 20
type = "docs"
description = "使用应用层传输安全（ALTS）进行 gRPC 认证的概述。"
isCJKLanguage = true
draft = false

+++

> 原文：[https://grpc.io/docs/languages/go/alts/](https://grpc.io/docs/languages/go/alts/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# ALTS authentication - ALTS 认证

An overview of gRPC authentication in Go using Application Layer Transport Security (ALTS).

​	使用应用层传输安全（ALTS）进行 gRPC 认证的概述。

### Overview

Application Layer Transport Security (ALTS) is a mutual authentication and transport encryption system developed by Google. It is used for securing RPC communications within Google’s infrastructure. ALTS is similar to mutual TLS but has been designed and optimized to meet the needs of Google’s production environments. For more information, take a look at the [ALTS whitepaper](https://cloud.google.com/security/encryption-in-transit/application-layer-transport-security).

​	应用层传输安全（Application Layer Transport Security, ALTS）是 Google 开发的一种相互认证和传输加密系统，用于保护 Google 基础设施内的 RPC 通信。ALTS 类似于双向 TLS，但专为满足 Google 生产环境需求而设计和优化。有关更多信息，请参阅 [ALTS 白皮书](https://cloud.google.com/security/encryption-in-transit/application-layer-transport-security)。

ALTS in gRPC has the following features:

​	gRPC 中的 ALTS 具有以下特性：

- Create gRPC servers & clients with ALTS as the transport security protocol.
  - 使用 ALTS 作为传输安全协议创建 gRPC 服务器和客户端。

- ALTS connections are end-to-end protected with privacy and integrity.
  - ALTS 连接提供端到端的隐私和完整性保护。

- Applications can access peer information such as the peer service account.
  - 应用程序可以访问对等方信息，例如对等服务账户。

- Client authorization and server authorization support.
  - 支持客户端授权和服务器授权。

- Minimal code changes to enable ALTS.
  - 启用 ALTS 所需的代码更改最少。


gRPC users can configure their applications to use ALTS as a transport security protocol with few lines of code.

​	gRPC 用户可以通过几行代码将其应用程序配置为使用 ALTS 作为传输安全协议。

Note that ALTS is fully functional if the application runs on [Compute Engine](https://cloud.google.com/compute) or [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine).

​	请注意，如果应用程序运行在 [Compute Engine](https://cloud.google.com/compute) 或 [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) 上，ALTS 功能完全可用。

### 使用 ALTS 传输安全协议的 gRPC 客户端 gRPC Client with ALTS Transport Security Protocol

gRPC clients can use ALTS credentials to connect to servers, as illustrated in the following code excerpt:

​	gRPC 客户端可以使用 ALTS 凭据连接到服务器，如以下代码片段所示：

```go
import (
  "google.golang.org/grpc"
  "google.golang.org/grpc/credentials/alts"
)

altsTC := alts.NewClientCreds(alts.DefaultClientOptions())
conn, err := grpc.NewClient(serverAddr, grpc.WithTransportCredentials(altsTC))
```

### 使用 ALTS 传输安全协议的 gRPC 服务器 gRPC Server with ALTS Transport Security Protocol

gRPC servers can use ALTS credentials to allow clients to connect to them, as illustrated next:

​	gRPC 服务器可以使用 ALTS 凭据允许客户端连接，如下所示：

```go
import (
  "google.golang.org/grpc"
  "google.golang.org/grpc/credentials/alts"
)

altsTC := alts.NewServerCreds(alts.DefaultServerOptions())
server := grpc.NewServer(grpc.Creds(altsTC))
```

### 服务器授权 Server Authorization

gRPC has built-in server authorization support using ALTS. A gRPC client using ALTS can set the expected server service accounts prior to establishing a connection. Then, at the end of the handshake, server authorization guarantees that the server identity matches one of the service accounts specified by the client. Otherwise, the connection fails.

​	gRPC 使用 ALTS 提供内置的服务器授权支持。使用 ALTS 的 gRPC 客户端可以在建立连接之前设置预期的服务器服务账户。在握手结束时，服务器授权会验证服务器身份是否与客户端指定的服务账户之一匹配，否则连接失败。

```go
import (
  "google.golang.org/grpc"
  "google.golang.org/grpc/credentials/alts"
)

clientOpts := alts.DefaultClientOptions()
clientOpts.TargetServiceAccounts = []string{expectedServerSA}
altsTC := alts.NewClientCreds(clientOpts)
conn, err := grpc.NewClient(serverAddr, grpc.WithTransportCredentials(altsTC))
```

### 客户端授权 Client Authorization

On a successful connection, the peer information (e.g., client’s service account) is stored in the [AltsContext](https://github.com/grpc/grpc/blob/master/src/proto/grpc/gcp/altscontext.proto). gRPC provides a utility library for client authorization check. Assuming that the server knows the expected client identity (e.g., `foo@iam.gserviceaccount.com`), it can run the following example codes to authorize the incoming RPC.

​	在连接成功后，对等方信息（例如客户端的服务账户）会存储在 [AltsContext](https://github.com/grpc/grpc/blob/master/src/proto/grpc/gcp/altscontext.proto) 中。gRPC 提供了一个实用库来检查客户端授权。假设服务器知道预期的客户端身份（例如 `foo@iam.gserviceaccount.com`），可以运行以下示例代码对传入的 RPC 进行授权。

```go
import (
  "google.golang.org/grpc"
  "google.golang.org/grpc/credentials/alts"
)

err := alts.ClientAuthorizationCheck(ctx, []string{"foo@iam.gserviceaccount.com"})
```
