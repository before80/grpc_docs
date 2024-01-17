+++
title = "Keepalive"
date = 2024-01-17T08:51:13+08:00
weight = 110
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/guides/keepalive/](https://grpc.io/docs/guides/keepalive/)

# Keepalive 保持活动

How to use HTTP/2 PING-based keepalives in gRPC.

​	如何在 gRPC 中使用基于 HTTP/2 PING 的保持活动。



### Overview 概述

HTTP/2 PING-based keepalives are a way to keep an HTTP/2 connection alive even when there is no data being transferred. This is done by periodically sending a [PING frame](https://httpwg.org/specs/rfc7540.html#PING) to the other end of the connection. HTTP/2 keepalives can improve performance and reliability of HTTP/2 connections, but it is important to configure the keepalive interval carefully.

​	基于 HTTP/2 PING 的保持活动是一种在没有数据传输时保持 HTTP/2 连接活动的方法。这是通过定期向连接的另一端发送 PING 帧来完成的。HTTP/2 保持活动可以提高 HTTP/2 连接的性能和可靠性，但重要的是要仔细配置保持活动间隔。

#### Note 注意

There is a related but separate concern called [Health Checking]. Health checking allows a server to signal whether a *service* is healthy while keepalive is only about the *connection*.
有一个相关但独立的问题称为 [运行状况检查]。运行状况检查允许服务器发出服务是否正常运行的信号，而保持活动仅与连接有关。

### Background 背景

[TCP keepalive](https://en.wikipedia.org/wiki/Keepalive#TCP_keepalive) is a well-known method of maintaining connections and detecting broken connections. When TCP keepalive was enabled, either side of the connection can send redundant packets. Once ACKed by the other side, the connection will be considered as good. If no ACK is received after repeated attempts, the connection is deemed broken.

​	TCP 保持活动是维护连接和检测断开连接的众所周知的方法。启用 TCP 保持活动后，连接的任一端都可以发送冗余数据包。一旦另一端确认，连接将被视为良好。如果在多次尝试后未收到确认，则连接被视为断开。

Unlike TCP keepalive, gRPC uses HTTP/2 which provides a mandatory [PING frame](https://httpwg.org/specs/rfc7540.html#PING) which can be used to estimate round-trip time, bandwidth-delay product, or test the connection. The interval and retry in TCP keepalive don’t quite apply to PING because the transport is reliable, so they’re replaced with timeout (equivalent to interval * retry) in gRPC PING-based keepalive implementation.

​	与 TCP 保活不同，gRPC 使用 HTTP/2，它提供了一个强制性的 PING 帧，可用于估算往返时间、带宽延迟乘积或测试连接。TCP 保活中的间隔和重试并不完全适用于 PING，因为传输是可靠的，因此在基于 gRPC PING 的保活实现中，它们被超时（相当于间隔 * 重试）所取代。

#### Note 注意

It’s not required for service owners to support keepalive. **Client authors must coordinate with service owners** for whether a particular client-side setting is acceptable. Service owners decide what they are willing to support, including whether they are willing to receive keepalives at all (If the service does not support keepalive, the first few keepalive pings will be ignored, and the server will eventually send a `GOAWAY` message with debug data equal to the ASCII code for `too_many_pings`).
服务所有者不必支持保活。客户端作者必须与服务所有者协调，以确定某个客户端设置是否可接受。服务所有者决定他们愿意支持什么，包括他们是否愿意接收保活（如果服务不支持保活，则会忽略前几个保活 ping，服务器最终会发送一个 `GOAWAY` 消息，其中调试数据等于 `too_many_pings` 的 ASCII 代码）。

### How configuring keepalive affects a call 配置保活如何影响调用

Keepalive is less likely to be triggered for unary RPCs with quick replies. Keepalive is primarily triggered when there is a long-lived RPC, which will fail if the keepalive check fails and the connection is closed.

​	对于具有快速回复的单向 RPC，不太可能触发保活。保活主要在存在长寿命 RPC 时触发，如果保活检查失败并且连接关闭，则该 RPC 将失败。

For streaming RPCs, if the connection is closed, any in-progress RPCs will fail. If a call is streaming data, the stream will also be closed and any data that has not yet been sent will be lost.

​	对于流式 RPC，如果连接关闭，任何正在进行的 RPC 都将失败。如果调用正在流式传输数据，流也将关闭，并且尚未发送的任何数据都将丢失。

#### Warning 警告

To avoid DDoSing, it’s important to take caution when setting the keepalive configurations. Thus, it is recommended to avoid enabling keepalive without calls and for clients to avoid configuring their keepalive much below one minute.
为了避免 DDoSing，在设置保持活动配置时务必小心。因此，建议避免在没有调用时启用保持活动，并且建议客户端不要将保持活动配置得远低于一分钟。

### Common situations where keepalives can be useful 保持活动可能派上用场的常见情况

gRPC HTTP/2 keepalives can be useful in a variety of situations, including but not limited to:

​	gRPC HTTP/2 保持活动在各种情况下都可能派上用场，包括但不限于：

- When sending data over a long-lived connection which might be considered as idle by proxy or load balancers.
  通过代理或负载平衡器可能视为处于空闲状态的长期连接发送数据时。
- When the network is less reliable (For example, mobile applications).
  网络不太可靠时（例如，移动应用程序）。
- When using a connection after a long period of inactivity.
  长时间不活动后使用连接时。

### Keepalive configuration specification 保持活动配置规范

| Options 选项                     | Availability 可用性              | Description 说明                                             | Client Default 客户端默认值          | Server Default 服务器默认值         |
| -------------------------------- | -------------------------------- | ------------------------------------------------------------ | ------------------------------------ | ----------------------------------- |
| `KEEPALIVE_TIME`                 | Client and Server 客户端和服务器 | The interval in milliseconds between PING frames. PING 帧之间的毫秒数间隔。 | INT_MAX (Disabled) INT_MAX（已禁用） | 7200000 (2 hours) 7200000（2 小时） |
| `KEEPALIVE_TIMEOUT`              | Client and Server 客户端和服务器 | The timeout in milliseconds for a PING frame to be acknowledged. If sender does not receive an acknowledgment within this time, it will close the connection. PING 帧被确认的超时时间（以毫秒为单位）。如果发送方在此时间内未收到确认，它将关闭连接。 | 20000 (20 seconds) 20000（20 秒）    | 20000 (20 seconds) 20000（20 秒）   |
| `KEEPALIVE_WITHOUT_CALLS`        | Client 客户端                    | Is it permissible to send keepalive pings from the client without any outstanding streams. 是否允许从客户端发送 keepalive ping 而没有任何未完成的流。 | 0 (false) 0（否）                    | N/A                                 |
| `PERMIT_KEEPALIVE_WITHOUT_CALLS` | Server 服务器                    | Is it permissible to send keepalive pings from the client without any outstanding streams. 是否允许从客户端发送 keepalive ping 而没有任何未完成的流。 | N/A                                  | 0 (false) 0（否）                   |
| `PERMIT_KEEPALIVE_TIME`          | Server 服务器                    | Minimum allowed time between a server receiving successive ping frames without sending any data/header frame. 服务器在不发送任何数据/头帧的情况下接收连续 ping 帧之间允许的最小时间。 | N/A                                  | 300000 (5 minutes) 300000（5 分钟） |
| `MAX_CONNECTION_IDLE`            | Server 服务器                    | Maximum time that a channel may have no outstanding rpcs, after which the server will close the connection. 通道在没有未完成的 RPC 的情况下可以存在的最长时间，之后服务器将关闭连接。 | N/A                                  | INT_MAX (Infinite) INT_MAX（无限）  |
| `MAX_CONNECTION_AGE`             | Server 服务器                    | Maximum time that a channel may exist. 通道可以存在的最长时间。 | N/A                                  | INT_MAX (Infinite) INT_MAX（无限）  |
| `MAX_CONNECTION_AGE_GRACE`       | Server 服务器                    | Grace period after the channel reaches its max age. 通道达到其最大生存期后的宽限期。 | N/A                                  | INT_MAX (Infinite) INT_MAX（无限）  |

#### Note 注意

Some languages may provide additional options, please refer to language examples and additional resource for more details.
某些语言可能提供其他选项，请参阅语言示例和其他资源了解更多详情。

### Language guides and examples 语言指南和示例

| Language 语言 | Example 示例                                                 | Documentation 文档                                           |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| C++           | [C++ Example C++ 示例](https://github.com/grpc/grpc/tree/master/examples/cpp/keepalive) | [C++ Documentation C++ 文档](https://github.com/grpc/grpc/blob/master/doc/keepalive.md) |
| Go            | [Go Example Go 示例](https://github.com/grpc/grpc-go/tree/master/examples/features/keepalive) | [Go Documentation Go 文档](https://github.com/grpc/grpc-go/blob/master/Documentation/keepalive.md) |
| Java          | [Java Example Java 示例](https://github.com/grpc/grpc-java/tree/master/examples/src/main/java/io/grpc/examples/keepalive) | [Java Documentation Java 文档](https://grpc.github.io/grpc-java/javadoc/io/grpc/ManagedChannelBuilder.html#keepAliveTime-long-java.util.concurrent.TimeUnit-) |
| Python        | [Python Example Python 示例](https://github.com/grpc/grpc/tree/master/examples/python/keep_alive) | [Python Documentation Python 文档](https://github.com/grpc/grpc/blob/master/doc/keepalive.md) |

### Additional Resources 其他资源

- [gRFC for Client-side Keepalive
  客户端保持活动状态的 gRFC](https://github.com/grpc/proposal/blob/master/A8-client-side-keepalive.md)
- [gRFC for Server-side Connection Management
  服务器端连接管理的 gRFC](https://github.com/grpc/proposal/blob/master/A9-server-side-conn-mgt.md)
- [Using gRPC for Long-lived and Streaming RPCs
  使用 gRPC 进行长寿命和流式 RPC](https://www.youtube.com/watch?v=Naonb2XD_2Q)
