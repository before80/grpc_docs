+++
title = "gRPC at VSCO"
date = 2024-11-19T11:35:00+08:00
weight = 410
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/blog/vsco/](https://grpc.io/blog/vsco/)
>
> 收录该文档的时间：`2024-11-19T11:35:00+08:00`

# gRPC at VSCO

By **Robert Sayre** (VSCO), **Melinda Lu** (VSCO) | Tuesday, September 06, 2016



Our guest post today comes from Robert Sayre and Melinda Lu of VSCO.

Founded in 2011, [VSCO](https://vsco.co/) is a community for expression—empowering people to create, discover and connect through images and words. VSCO is in the process of migrating their stack to gRPC.

In 2015, user growth forced VSCO down a familiar path. A monolithic PHP application in existence since the early days of the company was exhibiting performance problems and becoming difficult to maintain. We experimented with some smaller services in node.js, Go, and Java. At the same time, a larger messaging service for email, push messages, and in-app notifications was built in Go. Taking a first step away from JSON, we chose [Protocol Buffers](https://developers.google.com/protocol-buffers/) as the serialization format for this system.

Today, VSCO has largely settled on Go for new services. There are exceptions, particularly where a mature JVM solution is available for a given problem. Additionally, VSCO uses node.js for web applications, often with server-side [React](https://facebook.github.io/react/). Given that mix of languages, services, and some future data pipeline work detailed below, VSCO settled on gRPC and Protocol Buffers as the most practical solution for interprocess communication. A gradual migration from JSON over HTTP/1.1 APIs to gRPC over HTTP/2 is underway and going well. That said, there have been issues with the maturity of the PHP implementation relative to other languages.

Protocol buffers have been particularly valuable in building out our data ecosystem, where we rely on them to standardize and allow safe evolution of our data schemas in a language-agnostic way. As one example, we’ve built a Go service that feeds off our MySQL and MongoDB database replication logs and transforms backend database changes into a stream of immutable events in Kafka, with each row- or document-change event encoded as a protocol buffer. This database event stream allows us to add real-time data consumers as desired, without impacting production traffic and without having to coordinate with other systems. By processing all database events into protocol buffers en-route to Kafka, we can ensure that data is encoded in a uniform way that makes it easy to consume and use from multiple languages. Our implementation of [MySQL-binary-log](https://github.com/vsco/autobahn-binlog) and [Mongo-oplog](https://github.com/vsco/autobahn-oplog) tailers are available on GitHub.

Elsewhere in our data pipeline, we’ve begun using gRPC and protocol buffers to deliver behavioral events from our iOS and Android clients to a Go ingestion service, which then publishes these events to Kafka. To support this high-volume use case, we needed (1) a performant, fault-tolerant, language-agnostic RPC framework, (2) a way to ensure data compatibility as our product evolves, and (3) horizontally-scalable infrastructure. We’ve found gRPC, protocol buffers, and Go services running in Kubernetes a good fit for all three. As this was our first client-facing Go gRPC service, we did experience some new points of friction — in particular, load-balancer support and amenities like curl-like debugging have been lagging due to the youth of the HTTP/2 ecosystem. However, the ease of defining services with the gRPC IDL, using built-in architecture like interceptors, and scaling with Go have made the tradeoffs worthwhile.

As a first step in bringing gRPC to our mobile clients, we’ve shipped telemetry code in our iOS and Android apps. As of gRPC 1.0, this process is relatively straightforward. They only post events to our servers so far, and don’t do much with gRPC responses. The previous implementation was based on JSON, and our move to a single protocol buffer definition of our events uncovered a bunch of subtle bugs and differences between the clients.

One slight roadblock we ran into was the need for our clients to maintain compatibility with our JSON implementation as we ramp up, and for integration with vendor SDKs. This required a little bit of key-value coding on iOS, but it got more difficult on Android. We ended up having to write a protobuf compiler plugin to get the reflection features we needed while maintaining adequate performance. Drawing from that experience, we’ve made a concise [example protoc plugin](https://github.com/vsco/protoc-demo) built with [Bazel](https://bazel.io/) available on GitHub.

As more and more of our data becomes available in protocol buffer form, we plan to build upon this unified schema to expand our machine-learning and analytics systems. For example, we write our Kafka database replication streams to Amazon S3 as [Apache Parquet](https://parquet.apache.org/), an efficient columnar disk-storage format. Parquet has low-level support for protocol buffers, so we can use our existing data definitions to write optimized tables and do partial deserializations where desired.

From S3, we run computations on our data using Apache Spark, which can use our protocol buffer definitions to define types. We’re also building new machine-learning applications with [TensorFlow](https://www.tensorflow.org/). It uses protocol buffers natively and allows us to serve our models as gRPC services with [TensorFlow Serving](https://tensorflow.github.io/serving/).

So far, we’ve had good luck with gRPC and Protocol Buffers. They don’t eliminate every integration headache. However it’s easy to see how they help our engineers avoid writing a lot of boilerplate RPC code, while side-stepping the endless data-quality papercuts that come with looser serialization formats.
