+++
title = "gRPC - now with easy installation"
date = 2024-11-19T11:35:00+08:00
weight = 460
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/blog/installation/](https://grpc.io/blog/installation/)
>
> 收录该文档的时间：`2024-11-19T11:35:00+08:00`

# gRPC - now with easy installation

By **Lisa Carey** (Google) | Monday, April 04, 2016



Today we are happy to provide an update that significantly simplifies the getting started experience for gRPC.

- For most languages, **the gRPC runtime can now be installed in a single step via native package managers** such as `npm` for Node.js, `gem` for Ruby and `pip` for Python. Even though our Node, Ruby and Python runtimes are wrapped on gRPC’s C core, users now don’t need to explicitly pre-install the C core library as a package in most Linux distributions. We autofetch it for you :-).
- **For Java, we have simplified the steps needed to add gRPC support to your build tools** by providing plugins for Maven and Gradle. These let you easily depend on the core runtime to deploy or ship generated libraries into production environments.
- You can also use our Dockerfiles to use these updated packages - deploying microservices built on gRPC should now be a very simple experience.

The installation story is not yet complete: we are now focused on improving your development experience by packaging our protocol buffer plugins in the same way as the gRPC runtime. This will simplify code generation and setting up your development environment.

### Want to try it?

Here’s how to install the gRPC runtime today in all our supported languages:

| Language    | Platform            | Command                                                      |
| ----------- | ------------------- | ------------------------------------------------------------ |
| Node.js     | Linux, Mac, Windows | `npm install grpc`                                           |
| Python      | Linux, Mac, Windows | `pip install grpcio`                                         |
| Ruby        | Linux, Mac, Windows | `gem install grpc`                                           |
| PHP         | Linux, Mac, Windows | `pecl install grpc-beta`                                     |
| Go          | Linux, Mac, Windows | `go get google.golang.org/grpc`                              |
| Objective-C | Mac                 | Runtime source fetched automatically from GitHub by CocoaPods |
| C#          | Windows             | Install [gRPC NuGet package](https://www.nuget.org/packages/Grpc/) from your IDE (Visual Studio, Monodevelop, Xamarin Studio) |
| Java        | Linux, Mac, Windows | Use our [Maven and Gradle plugins](https://github.com/grpc/grpc-java/blob/master/README.md) that provide gRPC with [statically linked `boringssl`](https://github.com/grpc/grpc-java/blob/master/SECURITY.md#openssl-statically-linked-netty-tcnative-boringssl-static) |
| C++         | Linux, Mac, Windows | Currently requires [manual build and install](https://github.com/grpc/grpc/blob/v1.66.0/src/cpp/README.md) |

You can find out more about installation in our [quick start pages]({{< ref "/docs/Languages" >}}) and GitHub repositories. Do send us your feedback on our [mailing list](https://groups.google.com/g/grpc-io) or file issues on our issue tracker if you run into any problems.
