+++
title = "Google Cloud PubSub - with the power of gRPC!"
date = 2024-11-19T11:35:00+08:00
weight = 470
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/blog/pubsub/](https://grpc.io/blog/pubsub/)
>
> 收录该文档的时间：`2024-11-19T11:35:00+08:00`

# Google Cloud PubSub - with the power of gRPC!

By **Lisa Carey** (Google) | Thursday, March 24, 2016



[Google Cloud PubSub](https://cloud.google.com/pubsub/) is Google’s scalable real-time messaging service that lets users send and receive messages between independent applications. It’s an important part of Google Cloud Platform’s big data offering, and is used by customers worldwide to build their own robust, global services. However, until now, the only way to use the Cloud PubSub API was via JSON over HTTP. That’s all changed with the release of [PubSub gRPC alpha](https://cloud.google.com/blog/big-data/2016/03/announcing-grpc-alpha-for-google-cloud-pubsub). Now **users can access PubSub via gRPC** and benefit from all the advantages it brings.

[Alpha instructions and gRPC code](https://cloud.google.com/pubsub/grpc-overview) are now available for gRPC PubSub in Python and Java.

But what if you want to use this service now with gRPC in another language - C#, say, or Ruby? Once you have a Google account, with a little bit of extra work you can do that too! You can use the tools and the instructions on [our site](https://grpc.io/docs/) to generate and use your own gRPC client code from the PubSub service’s `.proto` file, available from [GitHub](https://github.com/googleapis/googleapis/blob/master/google/pubsub/v1/pubsub.proto).

[Read the full Google Cloud PubSub announcement](https://cloud.google.com/blog/big-data/2016/03/announcing-grpc-alpha-for-google-cloud-pubsub)

[Find out more about using Google Cloud PubSub](https://cloud.google.com/pubsub/docs)
