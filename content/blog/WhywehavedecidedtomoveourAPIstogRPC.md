+++
title = "Why we have decided to move our APIs to gRPC"
date = 2024-11-19T11:35:00+08:00
weight = 420
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/blog/vendasta/](https://grpc.io/blog/vendasta/)
>
> 收录该文档的时间：`2024-11-19T11:35:00+08:00`

# Why we have decided to move our APIs to gRPC

By **Dale Hopkins** (CTO, [Vendasta](https://vendasta.com/)) | Originally written by Dale Hopkins with additional content by Lisa Carey and others at Google
 **Guest post**
Monday, August 29, 2016



Vendasta started out 8 years ago as a point solution provider of products for small business. From the beginning we partnered with media companies and agencies who have armies of salespeople and existing relationships with those businesses to sell our software. It is estimated that over 30 million small businesses exist in the United States alone, so scalability of our SaaS solution was considered one of our top concerns from the beginning and it was the reason we started with [Google App Engine](https://cloud.google.com/appengine/) and Datastore. This solution worked really well for us as our system scaled from hundreds to hundreds of thousands of end users. We also scaled our offering from a point solution to an entire platform with multiple products and the tools for partners to manage their sales of those products during this time.

All throughout this journey Python GAE served our needs well. We exposed a number of APIs via HTTP + JSON for our partners to automate tasks and integrate their other systems with our products and platform. However, in 2016 we introduced the Vendasta Marketplace. This marked a major change to our offering, which depended heavily on having 3rd party vendors use our APIs to deliver their own products in our platform. This was a major change because our public APIs provide an upper-bound on 3rd-party applications, and made us realize that we really needed to make APIs that were amazing, not just good.

The first optimization that we started with was to use the Go programming language to build endpoints that handled higher throughput with lower latency than we could get with Python. On some APIs this made an incredible difference: we saw 50th percentile response times to drop from 1200 ms to 4 ms, and even more spectacularly 99th percentile response times drop from 30,000 ms to 12 ms! On other APIs we saw a much smaller, but still significant difference.

The second optimization we used was to replicate large portions of our Datastore data into ElasticSearch. ElasticSearch is a fundamentally different storage technology to Datastore, and is not a managed service, so it was a big leap for us. But this change allowed us to migrate almost all of our overnight batch-processing APIs to real-time APIs. We had tried BigQuery, but it’s query processing times meant that we couldn’t display things in real time. We had tried cloudSQL, but there was too much data for it to easily scale. We had tried the appengine Search API, but it has limitations with result sets over 10,000. We instead scaled up our ElasticSearch cluster using [Google Container Engine](https://cloud.google.com/container-engine/) and with it’s powerful aggregations and facet processing our needs were easily met. So with these first two solutions in place, we had made meaningful changes to the performance of our APIs.

The last optimization we made was to move our APIs to [gRPC](https://grpc.io/). This change was much more extensive than the others as it affected our clients. Like ElasticSearch, it represents a fundamentally different model with differing performance characteristics, but unlike ElasticSearch we found it to be a true superset: all of our usage scenarios were impacted positively by it.

The first benefit we saw from gRPC was the ability to move from publishing APIs and asking developers to integrate with them, to releasing SDKs and asking developers to copy-paste example code written in their language. This represents a really big benefit for people looking to integrate with our products, while not requiring us to hand-roll entire SDKs in the 5+ languages our partners and vendors use. It is important to note that we still write light wrappers over the generated gRPC SDKs to make them package-manager friendly, and to provide wrappers over the generated protobuf structures.

The second benefit we saw from gRPC was the ability to break free from the call-and-response architecture necessitated by HTTP + JSON. gRPC is built on top of HTTP/2, which allows for client-side and/or server-side streaming. In our use cases, this means we can lower the time to first display by streaming results as they become ready on the server (server-side streaming). We have also been investigating the potential to offer very flexible create endpoints that easily support bulk ingestion with bi-directional streaming, this would mean we would allow the client to asynchronously stream results, while the server would stream back statuses allowing for easy checkpoint operations while not slowing upload speeds to wait for confirmations. We feel that we are just starting to see the benefits from this feature as it opens up a totally new model for client-server interactions that just wasn’t possible with HTTP.

The third benefit was the switch from JSON to protocol buffers, which works very well with gRPC. This improves serialization and deserialization times; which is very significant to some of our APIs, but appreciated on all of them. The more important benefit comes from the explicit format specification of proto, meaning that clients receive typed objects rather than free-form JSON. Because of this, our clients can reap the benefits of auto-completion in their IDEs, type-safety if their language supports it, and enforced compatibility between clients and servers with differing versions.

The final benefit of gRPC was our ability to quickly spec endpoints. The proto format for both data and service definition greatly simplifies defining new endpoints and finally allows the succinct definition of endpoint contracts. This means we are much better able to communicate endpoint specifications between our development teams. gRPC means that for the first time at our company we are able to simultaneously develop the client and the server side of our APIs! This means our latency to produce new APIs with the accompanying SDKs has dropped dramatically. Combined with code generation, it allows us to truly develop clients and servers in parallel.

Our experience with gRPC has been positive, even though it does not eliminate the difficulty of providing endpoints to partners and vendors, and address all of our performance issues. However, it does make improvements to our endpoint performance, integration with those endpoints, and even in delivery of SDKs.
