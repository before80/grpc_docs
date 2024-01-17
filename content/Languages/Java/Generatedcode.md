+++
title = "Generated code"
date = 2024-01-17T08:51:13+08:00
weight = 40
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/java/generated-code/](https://grpc.io/docs/languages/java/generated-code/)

# Generated-code reference 生成代码参考





## Packages 包裹

For each service defined in a .proto file, the Java code generation produces a Java class. The class name is the service’s name suffixed by `Grpc`. The package for the generated code is specified in the .proto file using the `java_package` option.

​	对于在 .proto 文件中定义的每个服务，Java 代码生成会生成一个 Java 类。类名是服务名称，后缀为 `Grpc` 。生成代码的包在 .proto 文件中使用 `java_package` 选项指定。

For example, if `ServiceName` is defined in a .proto file containing the following:

​	例如，如果 `ServiceName` 在包含以下内容的 .proto 文件中定义：

```protobuf
package grpcexample;

option java_package = "io.grpc.examples";
```

Then the generated class will be `io.grpc.examples.ServiceNameGrpc`.

​	那么生成的类将是 `io.grpc.examples.ServiceNameGrpc` 。

If `java_package` is not specified, the generated class will use the `package` as specified in the .proto file. This should be avoided, as proto packages usually do not begin with a reversed domain name.

​	如果未指定 `java_package` ，生成的类将使用 .proto 文件中指定的 `package` 。应避免这种情况，因为 proto 包通常不以反向域名开头。

## Service Stub 服务存根

The generated Java code contains an inner abstract class suffixed with `ImplBase`, such as `ServiceNameImplBase`. This class defines one Java method for each method in the service definition. It is up to the service implementer to extend this class and implement the functionality of these methods. Without being overridden, the methods return an error to the client saying the method is unimplemented.

​	生成的 Java 代码包含一个内部抽象类，后缀为 `ImplBase` ，例如 `ServiceNameImplBase` 。此类为服务定义中的每个方法定义一个 Java 方法。服务实现者负责扩展此类并实现这些方法的功能。如果没有被覆盖，这些方法会向客户端返回一个错误，说明该方法尚未实现。

The signatures of the stub methods in `ServiceNameImplBase` vary depending on the type of RPCs it handles. There are four types of gRPC service methods: unary, server-streaming, client-streaming, and bidirectional-streaming.

​	 `ServiceNameImplBase` 中存根方法的签名会根据它处理的 RPC 类型而有所不同。gRPC 服务方法有四种类型：一元、服务器流式、客户端流式和双向流式。

### Unary 一元

The service stub signature for a unary RPC method `unaryExample`:

​	一元 RPC 方法的服务存根签名 `unaryExample` :

```java
public void unaryExample(
    RequestType request,
    StreamObserver<ResponseType> responseObserver)
```

### Server-streaming 服务器流式传输

The service stub signature for a server-streaming RPC method `serverStreamingExample`:

​	服务器流式传输 RPC 方法的服务存根签名 `serverStreamingExample` :

```java
public void serverStreamingExample(
    RequestType request,
    StreamObserver<ResponseType> responseObserver)
```

Notice that the signatures for unary and server-streaming RPCs are the same. A single `RequestType` is received from the client, and the service implementation sends its response(s) by invoking `responseObserver.onNext(ResponseType response)`.

​	请注意，一元和服务器流式传输 RPC 的签名是相同的。从客户端接收单个 `RequestType` ，服务实现通过调用 `responseObserver.onNext(ResponseType response)` 发送其响应。

### Client-streaming 客户端流式传输

The service stub signature for a client-streaming RPC method `clientStreamingExample`:

​	客户端流式传输 RPC 方法的服务存根签名 `clientStreamingExample` :

```java
public StreamObserver<RequestType> clientStreamingExample(
    StreamObserver<ResponseType> responseObserver)
```

### Bidirectional-streaming 双向流式传输

The service stub signature for a bidirectional-streaming RPC method `bidirectionalStreamingExample`:

​	双向流式传输 RPC 方法的服务存根签名 `bidirectionalStreamingExample` :

```java
public StreamObserver<RequestType> bidirectionalStreamingExample(
    StreamObserver<ResponseType> responseObserver)
```

The signatures for client and bidirectional-streaming RPCs are the same. Since the client can send multiple messages to the service, the service implementation is responsible for returning a `StreamObserver<RequestType>` instance. This `StreamObserver` is invoked whenever additional messages are received from the client.

​	客户端和双向流式传输 RPC 的签名是相同的。由于客户端可以向服务发送多条消息，因此服务实现负责返回 `StreamObserver<RequestType>` 实例。每当从客户端收到其他消息时，都会调用此 `StreamObserver` 。

## Client Stubs 客户端存根

The generated class also contains stubs for use by gRPC clients to call methods defined by the service. Each stub wraps a `Channel`, supplied by the user of the generated code. The stub uses this channel to send RPCs to the service.

​	生成的类还包含存根，供 gRPC 客户端用来调用服务定义的方法。每个存根都包装一个 `Channel` ，由生成的代码的用户提供。存根使用此通道向服务发送 RPC。

gRPC Java generates code for three types of stubs: asynchronous, blocking, and future. Each type of stub has a corresponding class in the generated code, such as `ServiceNameStub`, `ServiceNameBlockingStub`, and `ServiceNameFutureStub`.

​	gRPC Java 为三种类型的存根生成代码：异步、阻塞和将来。每种类型的存根在生成的代码中都有一个相应的类，例如 `ServiceNameStub` 、 `ServiceNameBlockingStub` 和 `ServiceNameFutureStub` 。

### Asynchronous Stub 异步存根

RPCs made via an asynchronous stub operate entirely through callbacks on `StreamObserver`.

​	通过异步存根进行的 RPC 完全通过 `StreamObserver` 上的回调操作。

The asynchronous stub contains one Java method for each method from the service definition.

​	异步存根包含一个 Java 方法，用于服务定义中的每个方法。

A new asynchronous stub is instantiated via the `ServiceNameGrpc.newStub(Channel channel)` static method.

​	通过 `ServiceNameGrpc.newStub(Channel channel)` 静态方法实例化新的异步存根。

#### Unary 一元

The asynchronous stub signature for a unary RPC method `unaryExample`:

​	一元 RPC 方法 `unaryExample` 的异步存根签名：

```java
public void unaryExample(
    RequestType request,
    StreamObserver<ResponseType> responseObserver)
```

#### Server-streaming 服务器流

The asynchronous stub signature for a server-streaming RPC method `serverStreamingExample`:

​	服务器流 RPC 方法 `serverStreamingExample` 的异步存根签名：

```java
public void serverStreamingExample(
    RequestType request,
    StreamObserver<ResponseType> responseObserver)
```

#### Client-streaming 客户端流式传输

The asynchronous stub signature for a client-streaming RPC method `clientStreamingExample`:

​	客户端流式传输 RPC 方法的异步存根签名 `clientStreamingExample` :

```java
public StreamObserver<RequestType> clientStreamingExample(
    StreamObserver<ResponseType> responseObserver)
```

#### Bidirectional-streaming 双向流式传输

The asynchronous stub signature for a bidirectional-streaming RPC method `bidirectionalStreamingExample`:

​	双向流式传输 RPC 方法的异步存根签名 `bidirectionalStreamingExample` :

```java
public StreamObserver<RequestType> bidirectionalStreamingExample(
    StreamObserver<ResponseType> responseObserver)
```

### Blocking Stub 阻塞存根

RPCs made through a blocking stub, as the name implies, block until the response from the service is available.

​	顾名思义，通过阻塞存根进行的 RPC 会阻塞，直到服务响应可用。

The blocking stub contains one Java method for each unary and server-streaming method in the service definition. Blocking stubs do not support client-streaming or bidirectional-streaming RPCs.

​	阻塞存根包含一个 Java 方法，用于服务定义中的每个单向和服务器流式传输方法。阻塞存根不支持客户端流式传输或双向流式传输 RPC。

A new blocking stub is instantiated via the `ServiceNameGrpc.newBlockingStub(Channel channel)` static method.

​	通过 `ServiceNameGrpc.newBlockingStub(Channel channel)` 静态方法实例化新的阻塞存根。

#### Unary 单向

The blocking stub signature for a unary RPC method `unaryExample`:

​	单向 RPC 方法的阻塞存根签名 `unaryExample` :

```java
public ResponseType unaryExample(RequestType request)
```

#### Server-streaming 服务器流式传输

The blocking stub signature for a server-streaming RPC method `serverStreamingExample`:

​	服务器流式传输 RPC 方法 `serverStreamingExample` 的阻塞存根签名：

```java
public Iterator<ResponseType> serverStreamingExample(RequestType request)
```

### Future Stub Future 存根

RPCs made via a future stub wrap the return value of the asynchronous stub in a `GrpcFuture<ResponseType>`, which implements the `com.google.common.util.concurrent.ListenableFuture` interface.

​	通过 Future 存根进行的 RPC 将异步存根的返回值包装在 `GrpcFuture<ResponseType>` 中，该值实现了 `com.google.common.util.concurrent.ListenableFuture` 接口。

The future stub contains one Java method for each unary method in the service definition. Future stubs do not support streaming calls.

​	Future 存根包含服务定义中的每个单向方法的一个 Java 方法。Future 存根不支持流式调用。

A new future stub is instantiated via the `ServiceNameGrpc.newFutureStub(Channel channel)` static method.

​	通过 `ServiceNameGrpc.newFutureStub(Channel channel)` 静态方法实例化新的 Future 存根。

#### Unary 单向

The future stub signature for a unary RPC method `unaryExample`:

​	单向 RPC 方法 `unaryExample` 的 Future 存根签名：

```java
public ListenableFuture<ResponseType> unaryExample(RequestType request)
```

## Codegen 代码生成

Typically the build system handles creation of the gRPC generated code.

​	通常，构建系统处理 gRPC 生成代码的创建。

For protobuf-based codegen, you can put your `.proto` files in the `src/main/proto` and `src/test/proto` directories along with an appropriate plugin.

​	对于基于 protobuf 的代码生成，您可以将 `.proto` 文件与适当的插件一起放在 `src/main/proto` 和 `src/test/proto` 目录中。

A typical [protobuf-maven-plugin](https://www.xolstice.org/protobuf-maven-plugin/) configuration for generating gRPC and Protocol Buffers code would look like the following:

​	用于生成 gRPC 和 Protocol Buffers 代码的典型 protobuf-maven-plugin 配置如下所示：

```xml
<build>
  <extensions>
    <extension>
      <groupId>kr.motd.maven</groupId>
      <artifactId>os-maven-plugin</artifactId>
      <version>1.4.1.Final</version>
    </extension>
  </extensions>
  <plugins>
    <plugin>
      <groupId>org.xolstice.maven.plugins</groupId>
      <artifactId>protobuf-maven-plugin</artifactId>
      <version>0.5.0</version>
      <configuration>
        <protocArtifact>com.google.protobuf:protoc:3.3.0:exe:${os.detected.classifier}</protocArtifact>
        <pluginId>grpc-java</pluginId>
        <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.4.0:exe:${os.detected.classifier}</pluginArtifact>
      </configuration>
      <executions>
        <execution>
          <goals>
            <goal>compile</goal>
            <goal>compile-custom</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

Eclipse and NetBeans users should also look at `os-maven-plugin`’s [IDE documentation](https://github.com/trustin/os-maven-plugin#issues-with-eclipse-m2e-or-other-ides).

​	Eclipse 和 NetBeans 用户还应查看 `os-maven-plugin` 的 IDE 文档。

A typical [protobuf-gradle-plugin](https://github.com/google/protobuf-gradle-plugin) configuration would look like the following:

​	典型的 protobuf-gradle-plugin 配置如下所示：

```gradle
apply plugin: 'java'
apply plugin: 'com.google.protobuf'

buildscript {
  repositories {
    mavenCentral()
  }
  dependencies {
    // ASSUMES GRADLE 2.12 OR HIGHER. Use plugin version 0.7.5 with earlier
    // gradle versions
    classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.0'
  }
}

protobuf {
  protoc {
    artifact = "com.google.protobuf:protoc:3.2.0"
  }
  plugins {
    grpc {
      artifact = 'io.grpc:protoc-gen-grpc-java:1.4.0'
    }
  }
  generateProtoTasks {
    all()*.plugins {
      grpc {}
    }
  }
}
```

Bazel developers can use the [`java_grpc_library`](https://github.com/grpc/grpc-java/blob/master/java_grpc_library.bzl) rule, typically as follows:

​	Bazel 开发人员可以使用 `java_grpc_library` 规则，通常如下所示：

```java
load("@grpc_java//:java_grpc_library.bzl", "java_grpc_library")

proto_library(
    name = "helloworld_proto",
    srcs = ["src/main/proto/helloworld.proto"],
)

java_proto_library(
    name = "helloworld_java_proto",
    deps = [":helloworld_proto"],
)

java_grpc_library(
    name = "helloworld_java_grpc",
    srcs = [":helloworld_proto"],
    deps = [":helloworld_java_proto"],
)
```

Android developers, see [Generating client code]({{< ref "/Platforms/Android/Java/Basicstutorial#generating-client-code" >}}) for reference.

​	Android 开发人员，请参阅生成客户端代码以供参考。

If you wish to invoke the protobuf plugin for gRPC Java directly, the command-line syntax is as follows:

​	如果您希望直接调用 gRPC Java 的 protobuf 插件，则命令行语法如下：

```sh
$ protoc --plugin=protoc-gen-grpc-java \
    --grpc-java_out="$OUTPUT_FILE" --proto_path="$DIR_OF_PROTO_FILE" "$PROTO_FILE"
```
