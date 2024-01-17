+++
title = "Health Checking"
date = 2024-01-17T08:51:13+08:00
weight = 100
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/guides/health-checking/](https://grpc.io/docs/guides/health-checking/)

# Health Checking 健康检查

Explains how gRPC servers expose a health checking service and how client can be configured to automatically check the health of the server it is connecting to.

​	说明 gRPC 服务器如何公开健康检查服务，以及如何配置客户端以自动检查其连接到的服务器的运行状况。



### Overview 概述

gRPC specifies a standard service API ([health/v1](https://github.com/grpc/grpc-proto/blob/master/grpc/health/v1/health.proto)) for performing health check calls against gRPC servers. An implementation of this service is provided, but you are responsible for updating the health status of your services.

​	gRPC 指定了一个标准服务 API (health/v1)，用于对 gRPC 服务器执行运行状况检查调用。提供了此服务的实现，但您负责更新服务的运行状况。在客户端，您可以让客户端自动与后端的运行状况服务通信。这允许客户端避免被视为不健康的那些服务。服务器端运行状况服务 gRPC 服务器上的运行状况检查服务支持两种操作模式：对 rpc 端点的单播调用对于集中式监控或负载平衡解决方案很有用，但无法扩展以支持不断进行运行状况检查的 gRPC 客户端群使用 rpc 端点进行流式运行状况更新由 gRPC 客户端中的客户端运行状况检查功能使用在服务器上启用运行状况检查服务涉及以下步骤：使用提供的运行状况检查库创建运行状况检查服务

On the client side you can have the client automatically communicate with the health services of your backends. This allows the client to avoid services that are considered unhealthy.

### The Server Side Health Service

The health check service on a gRPC server supports two modes of operation:

- Unary calls to the

   

  ```
  Check
  ```

   

  rpc endpoint

  - Useful for centralized monitoring or load balancing solutions, but does not scale to support a fleet of gRPC client constantly making health checks

- Streaming health updates by using the

   

  ```
  Watch
  ```

   

  rpc endpoint

  - Used by the client side health check feature in gRPC clients

Enabling the health check service on your server involves the following steps:

1. Use the provided health check library to create a health check service

2. Add the health check service to your server.
   将健康检查服务添加到您的服务器。

3. Notify the health check library when the health of one of your services changes.

   
   当其中一项服务的运行状况发生变化时，通知运行状况检查库。

   - `NOT_SERVING` if your service cannot accept requests at the moment
     `NOT_SERVING` 如果您的服务目前无法接受请求
   - `SERVING` if your service is open for business
     `SERVING` 如果您的服务已开放营业
   - If you don’t care about the health of individual services, you can use an empty string ("") to represent the health of your whole server.
     如果您不关心各个服务的运行状况，则可以使用空字符串 (“”) 来表示整个服务器的运行状况。

4. Make sure you inform the health check library about server shutdown so that it can notify all the connected clients.
   请务必告知运行状况检查库有关服务器关闭的信息，以便它可以通知所有已连接的客户端。

The exact details vary by language, see the **Language Support** section below.

​	具体细节因语言而异，请参阅下面的语言支持部分。

### Enabling Client Health Checking 启用客户端运行状况检查

A gRPC client can be configured to perform health checks against the servers it connects to by modifying the [service config](https://github.com/grpc/grpc/blob/master/doc/service_config.md) of the channel. E.g. to monitor the health of the `foo` service you would use (in JSON format):

​	可以通过修改频道的服务配置来将 gRPC 客户端配置为对连接到的服务器执行运行状况检查。例如，要监控 `foo` 服务的运行状况，您将使用（JSON 格式）：

```json
{
  "healthCheckConfig": {
    "serviceName": "foo"
  }
}
```

Note that if your server reports health for the empty string ("") service, signifying the health of the whole server, you can also use an empty string here.

​	请注意，如果您的服务器报告空字符串 (“”) 服务的运行状况，表示整个服务器的运行状况，您也可以在此处使用空字符串。

Enabling health checking changes some behavior around calling a server:

​	启用运行状况检查会改变调用服务器的一些行为：

- The client will additionally call the

   

  ```
  Watch
  ```

   

  RPC on the health check service when a connection is established

  
  当建立连接时，客户端还将在运行状况检查服务上调用 `Watch` RPC

  - If the call fails, retries will be made (with exponential backoff), unless the call fails with the status UNIMPLEMENTED, in which case health checking will be disabled.
    如果调用失败，将进行重试（采用指数退避），除非调用失败的状态为 UNIMPLEMENTED，在这种情况下，运行状况检查将被禁用。

- Requests won’t be sent until the health check service sends a healthy status for the service being called
  在运行状况检查服务为被调用服务发送运行状况良好状态之前，不会发送请求

- If a healthy service becomes unhealthy the client will no longer send requests for that service
  如果运行状况良好的服务变得不健康，客户端将不再为此服务发送请求

- The calls will resume if the service later becomes healthy
  如果服务稍后变得健康，则调用将恢复

- Some load balancing policies can choose to disable health checking if the feature does not make sense with the policy (e.g. `pick_first` does this)
  如果该功能与该策略不符，某些负载平衡策略可以选择禁用运行状况检查（例如， `pick_first` 会这样做）

More specifically, the state of the subchannel (that represents the physical connection to the server) goes through these states based on the health of the service it is connecting to.

​	更具体地说，子信道的状态（表示与服务器的物理连接）会根据其连接到的服务的运行状况经历这些状态。

![image-20240117094905190](./HealthChecking_img/image-20240117094905190.png)

Again, the specifics on how to enable client side health checking varies by language, see the examples in the **Language Support** section.

​	同样，有关如何启用客户端运行状况检查的具体信息因语言而异，请参阅语言支持部分中的示例。

### Language Support 语言支持

| Language 语言 | Example 示例                                                 |
| ------------- | ------------------------------------------------------------ |
| Java          | [Java example Java 示例](https://github.com/grpc/grpc-java/tree/master/examples/src/main/java/io/grpc/examples/healthservice) |
| Go            | [Go example Go 示例](https://github.com/grpc/grpc-go/tree/master/examples/features/health) |
| Python        | [Python example Python 示例](https://github.com/grpc/grpc/tree/master/examples/python/health_checking) |
| C++           | Not yet available 尚未提供                                   |
