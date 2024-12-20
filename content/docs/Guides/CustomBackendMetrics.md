+++
title = "Custom Backend Metrics"
date = 2024-11-19T10:19:42+08:00
weight = 40
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/guides/custom-backend-metrics/](https://grpc.io/docs/guides/custom-backend-metrics/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Custom Backend Metrics

A mechanism in the gRPC library that allows users to inject custom metrics at the gRPC server and consume at gRPC clients to make your custom load balancing algorithms.



### Overview

Simple load balancing decisions can be made by taking into account local or global knowledge of a backend’s load, for example CPU. More sophisticated load balancing decisions are possible with application specific knowledge, e.g. queue depth, or by combining multiple metrics.

The custom backend metrics feature exposes APIs to allow users to implement the metrics feedback in their LB policies.

### Use Cases

The feature is mainly for advanced use cases where a custom LB policy is used to route traffic more intelligently to a list of backend servers to improve the routing performance, e.g. a weighted round robin LB policy.

gRPC traditionally allows users to plug in their own load balancing policies, see [guide]({{< ref "/docs/Guides/CustomLoadBalancingPolicies" >}}). For xDS users, [custom load balancer](https://github.com/grpc/proposal/blob/master/A52-xds-custom-lb-policies.md) can be configured to select the custom LB policy.

### Metrics Reporting

Open Request Cost Aggregation ([ORCA](https://github.com/cncf/xds/blob/main/xds/data/orca/v3/orca_load_report.proto)) is an open standard for conveying backend metrics information. gRPC uses ORCA service and metrics standards and supports two metrics reporting mechanisms:

- Per-query metrics reporting: the backend server attaches the injected custom metrics in the trailing metadata when the corresponding RPC finishes. This is typically useful for short RPCs like unary calls.
- Out-of-band metrics reporting: the backend server periodically pushes metrics data, e.g. cpu and memory utilization, to the client. This is useful for all situations: unary calls, long RPCs in streaming calls, or no RPCs. However, out-of-band metrics reporting does not send query cost metrics. The metrics emission frequency is user-configurable, and this configuration resides in the custom load balancing policy.

The diagram shows the architecture where a user creates their own LB policy that implements backend metrics feedback.



![gRPC backend metrics diagram](CustomBackendMetrics_img/backend_metrics.svg)



### Implementation

For more details, please see gRPC [proposal A51](https://github.com/grpc/proposal/blob/master/A51-custom-backend-metrics.md).

### Language Support

| Language | Example                                                      |
| -------- | ------------------------------------------------------------ |
| Java     | [Java example](https://github.com/grpc/grpc-java/tree/master/examples/example-orca) |
| Go       | [Go example](https://github.com/grpc/grpc-go/tree/master/examples/features/orca) |
| C++      | Example upcoming                                             |
