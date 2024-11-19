+++
title = "Debugging"
date = 2024-11-19T10:19:42+08:00
weight = 80
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/guides/debugging/](https://grpc.io/docs/guides/debugging/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Debugging

Explains the debugging process of gRPC applications using grpcdebug



### Overview

[grpcdebug](https://github.com/grpc-ecosystem/grpcdebug) is a command line tool within the gRPC ecosystem designed to assist developers in debugging and troubleshooting gRPC services. grpcdebug fetches the internal states of the gRPC library from the application via gRPC protocol and provides a human-friendly UX to browse them. Currently, it supports [Channelz](https://github.com/grpc/proposal/blob/master/A14-channelz.md)/[Health](https://github.com/grpc/grpc/blob/master/src/proto/grpc/health/v1/health.proto) Checking/CSDS (aka. [admin services](https://github.com/grpc/proposal/blob/master/A38-admin-interface-api.md)). In other words, it can fetch statistics about how many RPCs have being sent or failed on a given gRPC channel, it can inspect address resolution results, it can dump the active xDS configuration that directs the routing of RPCs.

### Language examples

| Language | Example                                                      | Notes                                                        |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| C++      | [C++ Example](https://github.com/grpc/grpc/tree/master/examples/cpp/debugging#using-grpcdebug) |                                                              |
| Go       | [Go Example](https://github.com/grpc-ecosystem/grpcdebug?tab=readme-ov-file#quick-start) | [Go test server implementing admin services from grpcdebug docs](https://github.com/grpc-ecosystem/grpcdebug/tree/main/internal/testing/testserver) |
| Java     | [Java Example](https://github.com/grpc/grpc-java/tree/master/examples/example-debug#using-grpcdebug) |                                                              |

### References

- [grpcdebug installation](https://github.com/grpc-ecosystem/grpcdebug?tab=readme-ov-file#installation)
- [grpcdebug quick start](https://github.com/grpc-ecosystem/grpcdebug?tab=readme-ov-file#quick-start)
