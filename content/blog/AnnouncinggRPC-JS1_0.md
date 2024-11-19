+++
title = "Announcing gRPC-JS 1.0"
date = 2024-11-19T11:35:00+08:00
weight = 130
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/blog/grpc-js-1.0/](https://grpc.io/blog/grpc-js-1.0/)
>
> 收录该文档的时间：`2024-11-19T11:35:00+08:00`

# Announcing gRPC-JS 1.0

By [**Michael Lumish**](https://github.com/murgatroid99) | Monday, April 20, 2020



We are excited to announce the release of version 1.0 of [**gRPC-JS** (@grpc/grpc-js)](https://www.npmjs.com/package/@grpc/grpc-js), a pure-TypeScript reimplementation of the original [Node gRPC library, grpc](https://www.npmjs.com/package/grpc).

## Features

gRPC-JS supports the following features, which should cover most use cases:

- Clients
- Automatic reconnection
- Servers
- Streaming
- Metadata
- Partial compression support: clients can decompress response messages
- Pick first and round robin load balancing policies
- Client Interceptors
- Connection Keepalives
- HTTP Connect support (proxies)

## Should I use @grpc/grpc-js or grpc?

The original Node gRPC library ([grpc](https://www.npmjs.com/package/grpc)) will no longer receive feature updates and we plan to deprecate it in a year, so we recommend that you use gRPC-JS, [@grpc/grpc-js](https://www.npmjs.com/package/@grpc/grpc-js).

However, some advanced features haven’t been ported to gRPC-JS yet, such as full compression support or support for other load balancing policies. If you need one of these features, you should use the [grpc](https://www.npmjs.com/package/grpc) library, but open a [feature request](https://github.com/grpc/grpc-node/issues/new?template=feature_request.md) over gRPC-JS to let us know which features you are missing the most.

- [←Previous]({{< ref "/blog/KotlinmeetgRPC" >}})
- Next→
