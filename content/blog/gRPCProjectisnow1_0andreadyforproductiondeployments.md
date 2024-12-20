+++
title = "gRPC Project is now 1.0 and ready for production deployments"
date = 2024-11-19T11:35:00+08:00
weight = 430
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/blog/ga-announcement/](https://grpc.io/blog/ga-announcement/)
>
> 收录该文档的时间：`2024-11-19T11:35:00+08:00`

# gRPC Project is now 1.0 and ready for production deployments

By [**Varun Talwar**](https://cloud.google.com/) (Google) | Originally written by Varun Talwar with additional content by Kailash Sethuraman and others at Google.
Tuesday, August 23, 2016



Today, the gRPC project has reached a significant milestone with its [1.0 release](https://github.com/grpc/grpc/releases). Languages moving to 1.0 include C++, Java, Go, Node, Ruby, Python and C# across Linux, Windows, and Mac. Objective-C and Android Java support on iOS and Android is also moving to 1.0. The 1.0 release means that the core protocol and API surface are now stable with measured performance, stress tested and developers can rely on these APIs and deploy in production, they will follow semantic versioning from here.

We are very excited about the progress we have made so far and would like to thank all our users and contributors. First announced in March 2015 with [Square](https://corner.squareup.com/2015/02/grpc.html), gRPC is already being used in many open source projects like [etcd](https://github.com/coreos/etcd) from CoreOS, [containerd](https://github.com/docker/containerd) from Docker, [cockroachdb](https://github.com/cockroachdb/cockroach) from Cockroach Labs, and by many other companies like [Vendasta](https://vendasta.com/), [Netflix](https://github.com/Netflix/ribbon), [YikYak](http://yikyakapp.com/) and [Carbon 3d](http://carbon3d.com/). Outside of microservices, telecom giants like [Cisco](https://github.com/CiscoDevNet/grpc-getting-started), [Juniper](https://github.com/Juniper/open-nti), [Arista](https://github.com/aristanetworks/goarista), and Ciena, are building support for streaming telemetry and network configuration from their network devices using gRPC, as part of [OpenConfig](http://www.openconfig.net/) effort.

From the beta release, we have made significant strides in the areas of usability, interoperability, and performance measurement on the [road to 1.0](https://www.youtube.com/watch?v=_vfbVJ_u5mE). In most of the languages, the [installation of the gRPC runtime]({{< ref "/blog/gRPC-nowwitheasyinstallation" >}}) as well as setup of a development environment is a single command. Beyond installation, we have set up automated tests for gRPC across languages and RPC types in order to stress test our APIs and ensure interoperability. There is now a [performance dashboard](https://goo.gl/tHPEfD) available in the open to see latency and throughput for unary and streaming ping pong for various languages. Other measurements have shown significant gains from using gRPC/Protobuf instead of HTTP/JSON such as in [CoreOS blogpost](https://blog.gopheracademy.com/advent-2015/etcd-distributed-key-value-store-with-grpc-http2/) and in [Google Cloud PubSub testing](https://cloud.google.com/blog/big-data/2016/03/announcing-grpc-alpha-for-google-cloud-pubsub). In the coming months, we will invest a lot more in performance tuning.

Even within Google, we have seen Google cloud APIs like [BigTable](https://cloudplatform.googleblog.com/2015/07/A-Go-client-for-Google-Cloud-Bigtable.html), PubSub, [Speech](https://github.com/GoogleCloudPlatform/java-docs-samples/tree/master/speech/grpc), launch of a gRPC-based API surface leading to ease of use and performance benefits. Products like [Tensorflow](https://research.googleblog.com/2016/02/running-your-models-in-production-with.html) have effectively used gRPC for inter-process communication as well. Beyond usage, we are keen to see the contributor community grow with gRPC. We are already starting to see contributions around gRPC in meaningful ways in the [grpc-ecosystem](https://github.com/grpc-ecosystem) organization. We are very happy to see projects like [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) to enable users to serve REST clients with gRPC based services, [Polyglot](https://github.com/grpc-ecosystem/polyglot) to have a CLI for gRPC, [Prometheus monitoring](https://github.com/grpc-ecosystem/go-grpc-prometheus) of gRPC Services and work with [OpenTracing](https://github.com/grpc-ecosystem/grpc-opentracing). You can suggest and contribute projects to this organization [here](https://docs.google.com/a/google.com/forms/d/119zb79XRovQYafE9XKjz9sstwynCWcMpoJwHgZJvK74/edit). We look forward to working with the community to take the gRPC project to new heights.
