+++
title = "OAuth2"
date = 2024-01-17T08:51:13+08:00
weight = 20
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/languages/objective-c/oauth2/](https://grpc.io/docs/languages/objective-c/oauth2/)

# OAuth2





This example demonstrates how to use OAuth2 on gRPC to make authenticated API calls on behalf of a user.

​	此示例演示如何使用 gRPC 上的 OAuth2 代表用户进行身份验证的 API 调用。

By walking through it you’ll also learn how to use the Objective-C gRPC API to:

​	通过逐步操作，您还将学习如何使用 Objective-C gRPC API 来：

- Initialize and configure a remote call object before the RPC is started.
  在启动 RPC 之前初始化和配置远程调用对象。
- Set request metadata elements on a call, which are semantically equivalent to HTTP request headers.
  在调用中设置请求元数据元素，这些元素在语义上等同于 HTTP 请求标头。
- Read response metadata from a call, which is equivalent to HTTP response headers and trailers.
  从调用中读取响应元数据，这等同于 HTTP 响应标头和尾部。

It assumes you know the basics on how to make gRPC API calls using the Objective-C client library, as shown in [Basics tutorial]({{< ref "/Languages/Objective-C/Basicstutorial" >}}) and the [Introduction to gRPC]({{< ref "/WhatisgRPC/Introduction" >}}), and are familiar with OAuth2 concepts like *access token*.

​	它假定您知道如何使用 Objective-C 客户端库进行 gRPC API 调用，如基本教程和 gRPC 简介中所示，并且熟悉 OAuth2 概念，如访问令牌。

### Example code and setup 示例代码和设置

For the example source, see [gprc/examples/objective-c/auth_sample](https://github.com/grpc/grpc/tree/v1.60.0/examples/objective-c/auth_sample). To download the example, clone this repository by running the following commands:

​	有关示例源，请参阅 gprc/examples/objective-c/auth_sample。要下载示例，请通过运行以下命令克隆此存储库：

```sh
$ git clone -b v1.60.0 --depth 1 --shallow-submodules https://github.com/grpc/grpc
$ cd grpc
$ git submodule update --init
```

Then change your current directory to `examples/objective-c/auth_sample`:

​	然后将当前目录更改为 `examples/objective-c/auth_sample` ：

```sh
$ cd examples/objective-c/auth_sample
```

Our example is a simple application with two views. The first view lets a user sign in and out using the OAuth2 flow of Google’s [iOS SignIn library](https://developers.google.com/identity/sign-in/ios/). (Google’s library is used in this example because the test gRPC service we are going to call expects Google account credentials, but neither gRPC nor the Objective-C client library is tied to any specific OAuth2 provider). The second view makes a gRPC request to the test server, using the access token obtained by the first view.

​	我们的示例是一个具有两个视图的简单应用程序。第一个视图允许用户使用 Google 的 iOS SignIn 库的 OAuth2 流程登录和注销。（此示例中使用了 Google 的库，因为我们要调用的测试 gRPC 服务需要 Google 帐户凭据，但 gRPC 和 Objective-C 客户端库均不绑定到任何特定的 OAuth2 提供商）。第二个视图使用第一个视图获取的访问令牌向测试服务器发出 gRPC 请求。

#### Note 注意

OAuth2 libraries need the application to register and obtain an ID from the identity provider (in the case of this example app, Google). The app’s XCode project is configured using that ID, so you shouldn’t copy this project “as is” for your own app: it would result in your app being identified in the consent screen as “gRPC-AuthSample”, and not having access to real Google services. Instead, configure your own XCode project following the [instructions here](https://developers.google.com/identity/sign-in/ios/).
OAuth2 库需要应用程序注册并从身份提供者（在本示例应用程序中为 Google）获取 ID。应用程序的 XCode 项目使用该 ID 进行配置，因此您不应“按原样”将此项目复制到您自己的应用程序：这会导致您的应用程序在同意屏幕中被标识为“gRPC-AuthSample”，并且无法访问真正的 Google 服务。相反，请按照此处提供的说明配置您自己的 XCode 项目。

As with the other Objective-C examples, you also should have [CocoaPods](https://cocoapods.org/#install) installed, as well as the relevant tools to generate the client library code. You can obtain the latter by following [these setup instructions](https://github.com/grpc/homebrew-grpc).

​	与其他 Objective-C 示例一样，您还应该安装 CocoaPods，以及生成客户端库代码的相关工具。您可以按照这些设置说明获取后者。

### Try it out! 试一试！

To try the sample app, first have CocoaPods generate and install the client library for our .proto files:

​	要试用示例应用程序，首先让 CocoaPods 为我们的 .proto 文件生成并安装客户端库：

```sh
$ pod install
```

(This might have to compile OpenSSL, which takes around 15 minutes if CocoaPods doesn’t have it yet on your computer’s cache).

​	（这可能必须编译 OpenSSL，如果 CocoaPods 尚未在您计算机的缓存中，则大约需要 15 分钟）。

Finally, open the XCode workspace created by CocoaPods, and run the app.

​	最后，打开 CocoaPods 创建的 XCode 工作区，然后运行应用程序。

The first view, `SelectUserViewController.h/m`, asks you to sign in with your Google account, and to give the “gRPC-AuthSample” app the following permissions:

​	第一个视图 `SelectUserViewController.h/m` 会要求您使用您的 Google 帐户登录，并向“gRPC-AuthSample”应用程序授予以下权限：

- View your email address. 查看您的电子邮件地址。
- View your basic profile info.
  查看您的基本个人资料信息。
- “Test scope for access to the Zoo service”.
  “测试访问动物园服务的范围”。

This last permission, corresponding to the scope `https://www.googleapis.com/auth/xapi.zoo` doesn’t grant any real capability: it’s only used for testing. You can log out at any time.

​	此最后一个权限对应于范围 `https://www.googleapis.com/auth/xapi.zoo` ，不会授予任何实际功能：它仅用于测试。您可以随时注销。

The second view, `MakeRPCViewController.h/m`, makes a gRPC request to a test server at [https://grpc-test.sandbox.google.com](https://grpc-test.sandbox.google.com/), sending the access token along with the request. The test service simply validates the token and writes in its response which user it belongs to, and which scopes it gives access to. (The client application already knows those two values; it’s a way to verify that everything went as expected).

​	第二个视图 `MakeRPCViewController.h/m` 向 https://grpc-test.sandbox.google.com 上的测试服务器发出 gRPC 请求，同时发送访问令牌和请求。测试服务仅验证令牌，并在其响应中写入该令牌所属的用户以及它授予访问权限的范围。（客户端应用程序已知这两个值；这是一种验证一切是否按预期进行的方法）。

The next sections guide you step-by-step through how the gRPC call in `MakeRPCViewController` is performed. You can see the complete code in [MakeRPCViewController.m](https://github.com/grpc/grpc/blob/v1.60.0/examples/objective-c/auth_sample/MakeRPCViewController.m).

​	下一部分将逐步指导您如何执行 `MakeRPCViewController` 中的 gRPC 调用。您可以在 MakeRPCViewController.m 中看到完整代码。

### Create a call with access token 使用访问令牌创建调用

To make an authenticated call, first you need to initialize a `GRPCCallOptions` object and configure it with the access token.

​	要进行身份验证的调用，首先需要初始化一个 `GRPCCallOptions` 对象并使用访问令牌对其进行配置。

```objective-c
GRPCMutableCallOptions *options = [[GRPCMutableCallOptions alloc] init];
options.oauth2AccessToken = myAccessToken;
```

Then you need to create and start your call with this call options object. Assume you have a proto service definition like this:

​	然后，您需要使用此调用选项对象创建并启动您的调用。假设您有如下 proto 服务定义：

```protobuf
option objc_class_prefix = "AUTH";

service TestService {
  rpc UnaryCall(Request) returns (Response);
}
```

A `unaryCallWithMessage:responseHandler:callOptions:` method, with which you’re already familiar, is generated for the `AUTHTestService` class:

​	一个 `unaryCallWithMessage:responseHandler:callOptions:` 方法（您已经熟悉）为 `AUTHTestService` 类生成：

```objective-c
- (GRPCUnaryProtoRPC *)unaryCallWithMessage:(AUTHRequest *)message
                            responseHandler:(id<GRPCProtoResponseHandler>)responseHandler
                                callOptions:(GRPCCallOptions *)callOptions;
```

Use this method to generated the RPC object with your request options object:

​	使用此方法使用您的请求选项对象生成 RPC 对象：

```objective-c
GRPCUnaryProtoRPC *rpc = [client unaryCallWithMessage:myRequestMessage
                                      responseHandler:myResponseHandler
                                          callOptions:options];
```

You can then start the RPC represented by this object at any later time like this:

​	您可以在稍后随时像这样启动此对象所表示的 RPC：

```objective-c
[rpc start];
```

### An alternative way to provide access token 提供访问令牌的另一种方法

Rather than setting `oauth2AccessToken` option in `GRPCCallOptions` before the RPC object is created, an alternative approach allows users providing access token at call start time.

​	在创建 RPC 对象之前，无需在 `GRPCCallOptions` 中设置 `oauth2AccessToken` 选项，另一种方法允许用户在调用开始时提供访问令牌。

To use this approach, first create a class in your project that conforms to `GRPCAuthorizationProtocol` protocol.

​	要使用此方法，请首先在您的项目中创建一个符合 `GRPCAuthorizationProtocol` 协议的类。

```objective-c
@interface TokenProvider : NSObject<GRPCAuthorizationProtocol>
...
@end

@implementation TokenProvider

- (void)getTokenWithHandler:(void (^)(NSString* token))handler {
  ...
}

@end
```

When creating an RPC object, pass an instance of this class to call option `authTokenProvider`:

​	创建 RPC 对象时，将此类的实例传递给调用选项 `authTokenProvider` ：

```objective-c
GRPCMutableCallOptions *options = [[GRPCMutableCallOptions alloc] init];
options.authTokenProvider = [[TokenProvider alloc] init];
GRPCUnaryProtoCall *rpc = [client unaryCallWithMessage:myRequestMessage
                                       responseHandler:myResponseHandler
                                           callOptions:options] start];
[rpc start];
```

When the call starts, it will call the `TokenProvider` instance’s `getTokenWithHandler:` method with a callback `handler` and waits for the callback. The `TokenProvider` instance may call the handler at any time to provide the token for this call and resume the call process.

​	调用开始时，它将使用回调 `handler` 调用 `TokenProvider` 实例的 `getTokenWithHandler:` 方法并等待回调。 `TokenProvider` 实例可以随时调用处理程序来为此调用提供令牌并恢复调用过程。
