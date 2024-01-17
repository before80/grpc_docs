+++
title = "Quick start"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/platforms/web/quickstart/](https://grpc.io/docs/platforms/web/quickstart/)

# Quick start 快速入门

This guide gets you started with gRPC-Web with a simple working example.

​	本指南通过一个简单的示例帮助您开始使用 gRPC-Web。



### Prerequisites 先决条件

- Docker and `docker-compose` supporting [Docker Compose file version 3](https://docs.docker.com/compose/compose-file/compose-versioning). For installation instructions see [Install Compose](https://docs.docker.com/compose/install/#install-compose).

  ​	Docker 和 `docker-compose` 支持 Docker Compose 文件版本 3。有关安装说明，请参阅安装 Compose。

### Get the example code 获取示例代码

The example code is part of the [grpc-web](https://github.com/grpc/grpc-web) repo.

​	示例代码是 grpc-web 代码库的一部分。

1. [Download the repo as a zip file](https://github.com/grpc/grpc-web/archive/master.zip) and unzip it, or clone the repo:

   ​	以 zip 文件形式下载代码库并解压缩，或克隆代码库：

   ```sh
   $ git clone https://github.com/grpc/grpc-web
   ```

2. Change to the repo’s root directory:

   ​	更改到代码库的根目录：

   ```sh
   $ cd grpc-web
   ```

### Run an Echo example from your browser! 从浏览器运行一个 Echo 示例！

From the `grpc-web` directory:

​	从 `grpc-web` 目录：

1. Fetch required packages and tools:

   ​	获取所需的软件包和工具：

   ```sh
   $ docker-compose pull prereqs node-server envoy commonjs-client
   ```

   #### Note 注意

   Getting the following warning? You can ignore it for the purpose of running the example app:

   ​	收到以下警告？您可以忽略它以运行示例应用：

   ```nocode
   WARNING: Some service image(s) must be built from source
   ```

2. Launch services as background processes:

   ​	将服务作为后台进程启动：

   ```sh
   $ docker-compose up -d node-server envoy commonjs-client
   ```

3. From your browser: 
   ​	从您的浏览器：

   - Visit [localhost:8081/echotest.html](http://localhost:8081/echotest.html).
     访问 localhost:8081/echotest.html。
   - Enter a message, like “Hello”, in the text-input box.
     在文本输入框中输入一条消息，例如“Hello”。
   - Press the **Send** button. 按发送按钮。

   You’ll see your message echoed by the server below the input box.

   ​	您将在输入框下方的服务器中看到您的消息被回显。

Congratulations! You’ve just run a client-server application with gRPC.

​	恭喜！您刚刚使用 gRPC 运行了一个客户端-服务器应用程序。

Once you are done, shutdown the services that you launched earlier by running the following command:

​	完成后，通过运行以下命令关闭您之前启动的服务：

```sh
$ docker-compose down
```

### What is happening? 发生了什么？

This example app has three key components:

​	此示例应用程序具有三个关键组件：

1. `node-server` is a standard gRPC server, implemented in Node. This server listens at port `:9090`, and implements the app’s business logic (echoing client messages).
   `node-server` 是一个标准的 gRPC 服务器，在 Node 中实现。此服务器在端口 `:9090` 处侦听，并实现应用程序的业务逻辑（回显客户端消息）。
2. `envoy` is the Envoy proxy. It listens at `:8080` and forwards the browser’s gRPC-Web requests to port `:9090`.
   `envoy` 是 Envoy 代理。它在 `:8080` 处侦听，并将浏览器的 gRPC-Web 请求转发到端口 `:9090` 。
3. `commonjs-client`: this component generates the client stub class using the `protoc-gen-grpc-web` protoc plugin, compiles all the JS dependencies using `webpack`, and hosts the static content (`echotest.html` and `dist/main.js`) at port `:8081` using a simple web server. User messages entered from the webpage are sent to the Envoy proxy as gRPC-web requests.
   `commonjs-client` ：此组件使用 `protoc-gen-grpc-web` protoc 插件生成客户端存根类，使用 `webpack` 编译所有 JS 依赖项，并使用简单的 Web 服务器在端口 `:8081` 处托管静态内容（ `echotest.html` 和 `dist/main.js` ）。从网页输入的用户消息作为 gRPC-web 请求发送到 Envoy 代理。

### What’s next 下一步

- Work through the [Basics tutorial]({{< ref "/Platforms/Web/Basicstutorial" >}}).
  完成基础知识教程。
