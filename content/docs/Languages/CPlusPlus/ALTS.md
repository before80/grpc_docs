+++
title = "ALTS"
date = 2024-11-19T10:19:42+08:00
weight = 50
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/languages/cpp/alts/](https://grpc.io/docs/languages/cpp/alts/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# ALTS authentication

An overview of gRPC authentication in C++ using Application Layer Transport Security (ALTS).



### Overview

Application Layer Transport Security (ALTS) is a mutual authentication and transport encryption system developed by Google. It is used for securing RPC communications within Google’s infrastructure. ALTS is similar to mutual TLS but has been designed and optimized to meet the needs of Google’s production environments. For more information, take a look at the [ALTS whitepaper](https://cloud.google.com/security/encryption-in-transit/application-layer-transport-security).

ALTS in gRPC has the following features:

- Create gRPC servers & clients with ALTS as the transport security protocol.
- ALTS connections are end-to-end protected with privacy and integrity.
- Applications can access peer information such as the peer service account.
- Client authorization and server authorization support.
- Minimal code changes to enable ALTS.

gRPC users can configure their applications to use ALTS as a transport security protocol with few lines of code.

Note that ALTS is fully functional if the application runs on [Compute Engine](https://cloud.google.com/compute) or [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine).

### gRPC Client with ALTS Transport Security Protocol

gRPC clients can use ALTS credentials to connect to servers, as illustrated in the following code excerpt:

```cpp
#include <grpcpp/grpcpp.h>
#include <grpcpp/security/credentials.h>

using grpc::experimental::AltsCredentials;
using grpc::experimental::AltsCredentialsOptions;

auto creds = AltsCredentials(AltsCredentialsOptions());
std::shared_ptr<grpc::Channel> channel = CreateChannel(server_address, creds);
```

### gRPC Server with ALTS Transport Security Protocol

gRPC servers can use ALTS credentials to allow clients to connect to them, as illustrated next:

```cpp
#include <grpcpp/security/server_credentials.h>
#include <grpcpp/server.h>
#include <grpcpp/server_builder.h>

using grpc::experimental::AltsServerCredentials;
using grpc::experimental::AltsServerCredentialsOptions;

grpc::ServerBuilder builder;
builder.RegisterService(&service);
auto creds = AltsServerCredentials(AltsServerCredentialsOptions());
builder.AddListeningPort("[::]:<port>", creds);
std::unique_ptr<Server> server(builder.BuildAndStart());
```

### Server Authorization

gRPC has built-in server authorization support using ALTS. A gRPC client using ALTS can set the expected server service accounts prior to establishing a connection. Then, at the end of the handshake, server authorization guarantees that the server identity matches one of the service accounts specified by the client. Otherwise, the connection fails.

```cpp
#include <grpcpp/grpcpp.h>
#include <grpcpp/security/credentials.h>

using grpc::experimental::AltsCredentials;
using grpc::experimental::AltsCredentialsOptions;

AltsCredentialsOptions opts;
opts.target_service_accounts.push_back("expected_server_service_account1");
opts.target_service_accounts.push_back("expected_server_service_account2");
auto creds = AltsCredentials(opts);
std::shared_ptr<grpc::Channel> channel = CreateChannel(server_address, creds);
```

### Client Authorization

On a successful connection, the peer information (e.g., client’s service account) is stored in the [AltsContext](https://github.com/grpc/grpc/blob/master/src/proto/grpc/gcp/altscontext.proto). gRPC provides a utility library for client authorization check. Assuming that the server knows the expected client identity (e.g., `foo@iam.gserviceaccount.com`), it can run the following example codes to authorize the incoming RPC.

```cpp
#include <grpcpp/server_context.h>
#include <grpcpp/security/alts_util.h>

grpc::ServerContext* context;
grpc::Status status = experimental::AltsClientAuthzCheck(
    context->auth_context(), {"foo@iam.gserviceaccount.com"});
```
