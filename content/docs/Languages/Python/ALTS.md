+++
title = "ALTS"
date = 2024-11-19T10:19:42+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/languages/python/alts/](https://grpc.io/docs/languages/python/alts/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# ALTS authentication

An overview of gRPC authentication in Python using Application Layer Transport Security (ALTS).



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

```python
import grpc

channel_creds = grpc.alts_channel_credentials()
channel = grpc.secure_channel(address, channel_creds)
```

### gRPC Server with ALTS Transport Security Protocol

gRPC servers can use ALTS credentials to allow clients to connect to them, as illustrated next:

```python
import grpc

server = grpc.server(futures.ThreadPoolExecutor())
server_creds = grpc.alts_server_credentials()
server.add_secure_port(server_address, server_creds)
```
