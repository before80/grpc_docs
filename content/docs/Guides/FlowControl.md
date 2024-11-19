+++
title = "Flow Control"
date = 2024-11-19T10:19:42+08:00
weight = 100
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/guides/flow-control/](https://grpc.io/docs/guides/flow-control/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Flow Control

Explains what flow control is and how you can manually control it.



### Overview

Flow control is a mechanism to ensure that a receiver of messages does not get overwhelmed by a fast sender. Flow control prevents data loss, improves performance and increases reliability. It applies to streaming RPCs and is not relevant for unary RPCs. By default, gRPC handles the interactions with flow control for you, though some languages allow you to override the default behavior and take explicit control.

gRPC utilizes the underlying transport to detect when it is safe to send more data. As data is read on the receiving side, an acknowledgement is returned to the sender letting it know that the receiver has more capacity.

As needed, the gRPC framework will wait before returning from a write call. In gRPC, when a value is written to a stream, that does not mean that it has gone out over the network. Rather, that it has been passed to the framework which will now take care of the nitty gritty details of buffering it and sending it to the OS on its way over the network.

> Note
>
> The flow is the same for writing from a Server to a Client as when a Client writes to a Server

![image-20241119110209907](FlowControl_img/image-20241119110209907.png)

> Warning
>
> There is the potential for a deadlock if both the client and server are doing synchronous reads or using manual flow control and both try to do a lot of writing without doing any reads.

### Language Support

| Language | Example                                                      |
| -------- | ------------------------------------------------------------ |
| Java     | [Java Example](https://github.com/grpc/grpc-java/tree/master/examples/src/main/java/io/grpc/examples/manualflowcontrol) |
