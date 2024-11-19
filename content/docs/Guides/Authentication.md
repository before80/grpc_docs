+++
title = "认证"
date = 2024-11-19T10:19:42+08:00
weight = 1
type = "docs"
description = "gRPC 认证概述，包括内置认证机制以及如何集成自定义认证系统。"
isCJKLanguage = true
draft = false

+++

> 原文：[https://grpc.io/docs/guides/auth/](https://grpc.io/docs/guides/auth/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Authentication - 认证

An overview of gRPC authentication, including built-in auth mechanisms, and how to plug in your own authentication systems.

​	gRPC 认证概述，包括内置认证机制以及如何集成自定义认证系统。

### Overview

gRPC is designed to work with a variety of authentication mechanisms, making it easy to safely use gRPC to talk to other systems. You can use our supported mechanisms - SSL/TLS with or without Google token-based authentication - or you can plug in your own authentication system by extending our provided code.

​	gRPC 旨在支持多种认证机制，使得安全地使用 gRPC 与其他系统通信变得简单。您可以使用我们支持的机制（带或不带 Google 基于令牌的认证的 SSL/TLS），也可以通过扩展提供的代码集成自定义认证系统。

gRPC also provides a simple authentication API that lets you provide all the necessary authentication information as `Credentials` when creating a channel or making a call.

​	gRPC 还提供了一个简单的认证 API，允许您在创建通道或发起调用时通过 `Credentials` 提供所需的所有认证信息。

### 支持的认证机制 Supported auth mechanisms

The following authentication mechanisms are built-in to gRPC:

​	以下认证机制内置于 gRPC：

- **SSL/TLS**: gRPC has SSL/TLS integration and promotes the use of SSL/TLS to authenticate the server, and to encrypt all the data exchanged between the client and the server. Optional mechanisms are available for clients to provide certificates for mutual authentication.
  - **SSL/TLS**：gRPC 集成了 SSL/TLS，并推荐使用它来认证服务器以及加密客户端与服务器之间交换的所有数据。可选机制还支持客户端提供证书进行双向认证。

- **ALTS**: gRPC supports [ALTS](https://cloud.google.com/security/encryption-in-transit/application-layer-transport-security) as a transport security mechanism, if the application is running on [Compute Engine](https://cloud.google.com/compute) or [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine). For details, see one of the following language-specific pages: [ALTS in C++]({{< ref "/docs/Languages/CPlusPlus/ALTS" >}}), [ALTS in Go]({{< ref "/docs/Languages/Go/ALTS" >}}), [ALTS in Java]({{< ref "/docs/Languages/Java/ALTS" >}}), [ALTS in Python]({{< ref "/docs/Languages/Python/ALTS" >}}).
  - **ALTS**：如果应用程序运行在 [Compute Engine](https://cloud.google.com/compute) 或 [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) 上，gRPC 支持 [ALTS](https://cloud.google.com/security/encryption-in-transit/application-layer-transport-security) 作为传输安全机制。详细信息请参阅以下语言特定页面：[C++ 中的 ALTS]({{< ref "/docs/Languages/CPlusPlus/ALTS" >}})、[Go 中的 ALTS]({{< ref "/docs/Languages/Go/ALTS" >}})、[Java 中的 ALTS]({{< ref "/docs/Languages/Java/ALTS" >}})、[Python 中的 ALTS]({{< ref "/docs/Languages/Python/ALTS" >}})。

- **Token-based authentication with Google**: gRPC provides a generic mechanism (described below) to attach metadata based credentials to requests and responses. Additional support for acquiring access tokens (typically OAuth2 tokens) while accessing Google APIs through gRPC is provided for certain auth flows: you can see how this works in our code examples below. In general this mechanism must be used *as well as* SSL/TLS on the channel - Google will not allow connections without SSL/TLS, and most gRPC language implementations will not let you send credentials on an unencrypted channel.
  - **基于令牌的 Google 认证**：gRPC 提供一种通用机制（如下所述），可以将元数据凭据附加到请求和响应中。在通过 gRPC 访问 Google API 时，还为某些认证流程提供了获取访问令牌（通常是 OAuth2 令牌）的额外支持。此机制通常必须与通道上的 SSL/TLS 一起使用——Google 不允许没有 SSL/TLS 的连接，大多数 gRPC 语言实现也不允许在未加密的通道上发送凭据。

> Warning
>
> Google credentials should only be used to connect to Google services. Sending a Google issued OAuth2 token to a non-Google service could result in this token being stolen and used to impersonate the client to Google services.
>
> ​	Google 凭据应仅用于连接 Google 服务。将 Google 发布的 OAuth2 令牌发送到非 Google 服务可能导致令牌被盗用，并用于冒充客户端访问 Google 服务。

### 认证 API - Authentication API

gRPC provides a simple authentication API based around the unified concept of Credentials objects, which can be used when creating an entire gRPC channel or an individual call.

​	gRPC 提供了一个基于 `Credentials` 对象的简单认证 API，可以用于创建整个 gRPC 通道或单个调用。

#### 凭据类型 Credential types

Credentials can be of two types:

​	凭据可以分为两种类型：

- **Channel credentials**, which are attached to a `Channel`, such as SSL credentials.
  - **通道凭据**：附加到 `Channel`，例如 SSL 凭据。

- **Call credentials**, which are attached to a call (or `ClientContext` in C++).
  - **调用凭据**：附加到调用（或 C++ 中的 `ClientContext`）。


You can also combine these in a `CompositeChannelCredentials`, allowing you to specify, for example, SSL details for the channel along with call credentials for each call made on the channel. A `CompositeChannelCredentials` associates a `ChannelCredentials` and a `CallCredentials` to create a new `ChannelCredentials`. The result will send the authentication data associated with the composed `CallCredentials` with every call made on the channel.

​	您还可以通过 `CompositeChannelCredentials` 组合这两种凭据，例如在通道上指定 SSL 细节，同时为通道上的每次调用设置调用凭据。`CompositeChannelCredentials` 将 `ChannelCredentials` 和 `CallCredentials` 关联起来，创建一个新的 `ChannelCredentials`。结果是将组合的 `CallCredentials` 的认证数据发送到通道上的每次调用。

For example, you could create a `ChannelCredentials` from an `SslCredentials` and an `AccessTokenCredentials`. The result when applied to a `Channel` would send the appropriate access token for each call on this channel.

​	例如，您可以从 `SslCredentials` 和 `AccessTokenCredentials` 创建 `ChannelCredentials`。当应用于通道时，结果将为此通道上的每次调用发送相应的访问令牌。

Individual `CallCredentials` can also be composed using `CompositeCallCredentials`. The resulting `CallCredentials` when used in a call will trigger the sending of the authentication data associated with the two `CallCredentials`.

​	个别的 `CallCredentials` 也可以通过 `CompositeCallCredentials` 组合。在调用中使用时，组合的 `CallCredentials` 会触发与两个 `CallCredentials` 相关的认证数据发送。

#### 使用客户端 SSL/TLS - Using client-side SSL/TLS

Now let’s look at how `Credentials` work with one of our supported auth mechanisms. This is the simplest authentication scenario, where a client just wants to authenticate the server and encrypt all data. The example is in C++, but the API is similar for all languages: you can see how to enable SSL/TLS in more languages in our Examples section below.

​	以下展示了 `Credentials` 如何与一种支持的认证机制配合工作。这是最简单的认证场景，客户端只需认证服务器并加密所有数据。示例使用 C++，但其他语言的 API 类似：可在下方示例部分中查看如何在更多语言中启用 SSL/TLS。

```cpp
// Create a default SSL ChannelCredentials object.
// 创建默认 SSL 通道凭据对象。
auto channel_creds = grpc::SslCredentials(grpc::SslCredentialsOptions());
// Create a channel using the credentials created in the previous step.
// 使用上一步骤中创建的凭据创建通道。
auto channel = grpc::CreateChannel(server_name, channel_creds);
// Create a stub on the channel.
// 在通道上创建存根。
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
// Make actual RPC calls on the stub.
// 在存根上进行实际 RPC 调用。
grpc::Status s = stub->sayHello(&context, *request, response);
```

For advanced use cases such as modifying the root CA or using client certs, the corresponding options can be set in the `SslCredentialsOptions` parameter passed to the factory method.

​	对于更高级的用例，例如修改根 CA 或使用客户端证书，可以在传递给工厂方法的 `SslCredentialsOptions` 参数中设置相应选项。

> Note
>
> Non-POSIX-compliant systems (such as Windows) need to specify the root certificates in `SslCredentialsOptions`, since the defaults are only configured for POSIX filesystems.
>
> ​	非 POSIX 兼容系统（如 Windows）需要在 `SslCredentialsOptions` 中指定根证书，因为默认配置仅适用于 POSIX 文件系统。

#### 使用基于 OAuth 令牌的认证 Using OAuth token-based authentication

OAuth 2.0 Protocol is the industry-standard protocol for authorization. It enables websites or applications to obtain limited access to user accounts using OAuth tokens.

​	OAuth 2.0 是行业标准的认证协议，允许网站或应用程序通过 OAuth 令牌获得对用户账户的有限访问权限。

gRPC offers a set of simple APIs to integrate OAuth 2.0 into applications, streamlining authentication.

​	gRPC 提供了一组简单的 API，用于将 OAuth 2.0 集成到应用程序中，从而简化认证流程。

At a high level, using OAuth token-based authentication includes 3 steps:

​	高层次上，使用基于 OAuth 令牌的认证包括三个步骤：

1. Get or generate an OAuth token on client side. 客户端生成或获取 OAuth 令牌。
   - You can generate Google-specific tokens following instructions below.
     - 可按照以下说明生成 Google 特定令牌。
2. Create credentials with the OAuth token. 使用 OAuth 令牌创建凭据。
   - OAuth token is always part of per-call credentials, you can also attach the per-call credentials to some channel credentials.
     - OAuth 令牌始终是每次调用的凭据，也可以附加到某些通道凭据。
   - The token will be sent to server, normally as part of HTTP Authorization header.
     - 通常令牌作为 HTTP Authorization 标头的一部分发送到服务器。
3. Server side verifies the token. 服务器验证令牌。
   - In most implementations, the validation is done using a server side interceptor.
     - 在大多数实现中，验证通过服务器端拦截器完成。

For details of how to use OAuth token in different languages, please refer to our examples below.

​	有关在不同语言中如何使用 OAuth 令牌的详细信息，请参考下方的示例。

#### 使用 Google 基于令牌的认证 Using Google token-based authentication

gRPC applications can use a simple API to create a credential that works for authentication with Google in various deployment scenarios. Again, our example is in C++ but you can find examples in other languages in our Examples section.

​	gRPC 应用程序可以通过简单的 API 创建适用于 Google 认证的凭据，并支持各种部署场景。同样，示例使用 C++，但您可以在下方的示例部分找到其他语言的示例。

```cpp
auto creds = grpc::GoogleDefaultCredentials();
// Create a channel, stub and make RPC calls (same as in the previous example)
// 创建通道、存根并进行 RPC 调用（与前一示例相同）
auto channel = grpc::CreateChannel(server_name, creds);
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
grpc::Status s = stub->sayHello(&context, *request, response);
```

This channel credentials object works for applications using Service Accounts as well as for applications running in [Google Compute Engine (GCE)](https://cloud.google.com/compute/). In the former case, the service account’s private keys are loaded from the file named in the environment variable `GOOGLE_APPLICATION_CREDENTIALS`. The keys are used to generate bearer tokens that are attached to each outgoing RPC on the corresponding channel.

​	此通道凭据对象适用于使用服务账户的应用程序，以及运行在 [Google Compute Engine (GCE)](https://cloud.google.com/compute/) 上的应用程序。在前一种情况下，服务账户的私钥会从环境变量 `GOOGLE_APPLICATION_CREDENTIALS` 指定的文件中加载。这些密钥用于生成附加到相应通道上的每个传出 RPC 的承载令牌。

For applications running in GCE, a default service account and corresponding OAuth2 scopes can be configured during VM setup. At run-time, this credential handles communication with the authentication systems to obtain OAuth2 access tokens and attaches them to each outgoing RPC on the corresponding channel.

​	对于运行在 GCE 上的应用程序，可以在虚拟机设置期间配置默认服务账户及相应的 OAuth2 范围。在运行时，该凭据负责与认证系统通信以获取 OAuth2 访问令牌，并将其附加到相应通道上的每个传出 RPC。

#### 扩展 gRPC 以支持其他认证机制 Extending gRPC to support other authentication mechanisms

The Credentials plugin API allows developers to plug in their own type of credentials. This consists of:

​	凭据插件 API 允许开发者集成自定义类型的凭据。这包括：

- The `MetadataCredentialsPlugin` abstract class, which contains the pure virtual `GetMetadata` method that needs to be implemented by a sub-class created by the developer.
  - `MetadataCredentialsPlugin` 抽象类，其中包含需要由开发者创建的子类实现的纯虚函数 `GetMetadata`。

- The `MetadataCredentialsFromPlugin` function, which creates a `CallCredentials` from the `MetadataCredentialsPlugin`.
  - `MetadataCredentialsFromPlugin` 函数，用于从 `MetadataCredentialsPlugin` 创建 `CallCredentials`。


Here is example of a simple credentials plugin which sets an authentication ticket in a custom header.

​	以下是一个简单的凭据插件示例，它在自定义头部中设置一个认证票据。

```cpp
class MyCustomAuthenticator : public grpc::MetadataCredentialsPlugin {
 public:
  MyCustomAuthenticator(const grpc::string& ticket) : ticket_(ticket) {}

  grpc::Status GetMetadata(
      grpc::string_ref service_url, grpc::string_ref method_name,
      const grpc::AuthContext& channel_auth_context,
      std::multimap<grpc::string, grpc::string>* metadata) override {
    metadata->insert(std::make_pair("x-custom-auth-ticket", ticket_));
    return grpc::Status::OK;
  }

 private:
  grpc::string ticket_;
};

auto call_creds = grpc::MetadataCredentialsFromPlugin(
    std::unique_ptr<grpc::MetadataCredentialsPlugin>(
        new MyCustomAuthenticator("super-secret-ticket")));
```

A deeper integration can be achieved by plugging in a gRPC credentials implementation at the core level. gRPC internals also allow switching out SSL/TLS with other encryption mechanisms.

​	通过在核心级别集成 gRPC 凭据实现，可以实现更深度的集成。gRPC 内部还允许将 SSL/TLS 替换为其他加密机制。

### 语言指南与示例 Language guides and examples

These authentication mechanisms will be available in all gRPC’s supported languages. The following table links to examples demonstrating authentication and authorization in various languages.

​	这些认证机制可用于 gRPC 支持的所有语言。下表链接了不同语言中展示认证和授权的示例。

| Language | Example                                                      | Documentation                                                |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| C++      | N/A                                                          | N/A                                                          |
| Go       | [Go Example](https://github.com/grpc/grpc-go/tree/master/examples/features/encryption) | [Go Documentation](https://github.com/grpc/grpc-go/tree/master/examples/features/encryption#encryption) |
| Java     | [Java Example TLS](https://github.com/grpc/grpc-java/tree/master/examples/example-tls) ([Java Example ATLS](https://github.com/grpc/grpc-java/tree/master/examples/example-alts)) | [Java Documentation](https://github.com/grpc/grpc-java/tree/master/examples/example-tls#hello-world-example-with-tls) |
| Python   | [Python Example](https://github.com/grpc/grpc/tree/master/examples/python/auth) | [Python Documentation](https://github.com/grpc/grpc/tree/master/examples/python/auth#authentication-extension-example-in-grpc-python) |

### 基于 OAuth 令牌的认证语言指南和示例 Language guides and examples for OAuth token-based authentication

The following table links to examples demonstrating OAuth token-based authentication and authorization in various languages.

​	下表链接了不同语言中展示基于 OAuth 令牌的认证和授权的示例。

| Language | Example                                                      | Documentation                                                |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| C++      | N/A                                                          | N/A                                                          |
| Go       | [Go OAuth Example](https://github.com/grpc/grpc-go/tree/master/examples/features/authentication#authentication) | [Go OAuth Documentation](https://github.com/grpc/grpc-go/tree/master/examples/features/authentication#oauth2) |
| Java     | [Java OAuth Example](https://github.com/grpc/grpc-java/tree/master/examples/example-oauth#authentication-example) | [Java OAuth Documentation](https://github.com/grpc/grpc-java/tree/master/examples/example-oauth) |
| Python   | [Python OAuth Example](https://github.com/grpc/grpc/blob/master/examples/python/auth/token_based_auth_client.py) | [Python OAuth Documentation](https://github.com/grpc/grpc/tree/master/examples/python/auth#token-based-authentication) |

### 其他示例 Additional Examples

The following sections demonstrate how authentication and authorization features described above appear in other languages not listed above.

​	以下部分展示了其他未列出的语言中认证和授权特性如何实现。

#### Ruby

##### 基础案例 - 无加密或认证 Base case - no encryption or authentication

```ruby
stub = Helloworld::Greeter::Stub.new('localhost:50051', :this_channel_is_insecure)
...
```

##### 使用服务器认证的 SSL/TLS - With server authentication SSL/TLS

```ruby
creds = GRPC::Core::ChannelCredentials.new(load_certs)  # load_certs typically loads a CA roots file
stub = Helloworld::Greeter::Stub.new('myservice.example.com', creds)
```

##### 使用 Google 认证 Authenticate with Google

```ruby
require 'googleauth'  # from http://www.rubydoc.info/gems/googleauth/0.1.0
...
ssl_creds = GRPC::Core::ChannelCredentials.new(load_certs)  # load_certs typically loads a CA roots file
authentication = Google::Auth.get_application_default()
call_creds = GRPC::Core::CallCredentials.new(authentication.updater_proc)
combined_creds = ssl_creds.compose(call_creds)
stub = Helloworld::Greeter::Stub.new('greeter.googleapis.com', combined_creds)
```

#### Node.js

##### 基础案例 - 无加密/认证 Base case - No encryption/authentication

```js
var stub = new helloworld.Greeter('localhost:50051', grpc.credentials.createInsecure());
```

##### 使用服务器认证的 SSL/TLS - With server authentication SSL/TLS

```js
const root_cert = fs.readFileSync('path/to/root-cert');
const ssl_creds = grpc.credentials.createSsl(root_cert);
const stub = new helloworld.Greeter('myservice.example.com', ssl_creds);
```

##### 使用 Google 认证 Authenticate with Google

```js
// Authenticating with Google
// 使用 Google 认证
var GoogleAuth = require('google-auth-library'); // from https://www.npmjs.com/package/google-auth-library
...
var ssl_creds = grpc.credentials.createSsl(root_certs);
(new GoogleAuth()).getApplicationDefault(function(err, auth) {
  var call_creds = grpc.credentials.createFromGoogleCredential(auth);
  var combined_creds = grpc.credentials.combineChannelCredentials(ssl_creds, call_creds);
  var stub = new helloworld.Greeter('greeter.googleapis.com', combined_credentials);
});
```

##### 使用 Google OAuth2 令牌进行认证（遗留方法） Authenticate with Google using OAuth2 token (legacy approach)

```js
var GoogleAuth = require('google-auth-library'); // from https://www.npmjs.com/package/google-auth-library
...
var ssl_creds = grpc.Credentials.createSsl(root_certs); // load_certs typically loads a CA roots file - load_certs 通常加载 CA 根证书文件
var scope = 'https://www.googleapis.com/auth/grpc-testing';
(new GoogleAuth()).getApplicationDefault(function(err, auth) {
  if (auth.createScopeRequired()) {
    auth = auth.createScoped(scope);
  }
  var call_creds = grpc.credentials.createFromGoogleCredential(auth);
  var combined_creds = grpc.credentials.combineChannelCredentials(ssl_creds, call_creds);
  var stub = new helloworld.Greeter('greeter.googleapis.com', combined_credentials);
});
```

##### 使用服务器认证的 SSL/TLS 和自定义头部令牌 With server authentication SSL/TLS and a custom header with token

```js
const rootCert = fs.readFileSync('path/to/root-cert');
const channelCreds = grpc.credentials.createSsl(rootCert);
const metaCallback = (_params, callback) => {
    const meta = new grpc.Metadata();
    meta.add('custom-auth-header', 'token');
    callback(null, meta);
}
const callCreds = grpc.credentials.createFromMetadataGenerator(metaCallback);
const combCreds = grpc.credentials.combineChannelCredentials(channelCreds, callCreds);
const stub = new helloworld.Greeter('myservice.example.com', combCreds);
```

#### PHP

##### 基础案例 - 无加密/认证 Base case - No encryption/authorization

```php
$client = new helloworld\GreeterClient('localhost:50051', [
    'credentials' => Grpc\ChannelCredentials::createInsecure(),
]);
```

##### 使用服务器认证的 SSL/TLS - With server authentication SSL/TLS

```php
$client = new helloworld\GreeterClient('myservice.example.com', [
    'credentials' => Grpc\ChannelCredentials::createSsl(file_get_contents('roots.pem')),
]);
```

##### 使用 Google 认证 Authenticate with Google

```php
function updateAuthMetadataCallback($context)
{
    $auth_credentials = ApplicationDefaultCredentials::getCredentials();
    return $auth_credentials->updateMetadata($metadata = [], $context->service_url);
}
$channel_credentials = Grpc\ChannelCredentials::createComposite(
    Grpc\ChannelCredentials::createSsl(file_get_contents('roots.pem')),
    Grpc\CallCredentials::createFromPlugin('updateAuthMetadataCallback')
);
$opts = [
  'credentials' => $channel_credentials
];
$client = new helloworld\GreeterClient('greeter.googleapis.com', $opts);
```

##### 使用 Google OAuth2 令牌进行认证（遗留方法） Authenticate with Google using OAuth2 token (legacy approach)

```php
// the environment variable "GOOGLE_APPLICATION_CREDENTIALS" needs to be set
$scope = "https://www.googleapis.com/auth/grpc-testing";
$auth = Google\Auth\ApplicationDefaultCredentials::getCredentials($scope);
$opts = [
  'credentials' => Grpc\Credentials::createSsl(file_get_contents('roots.pem'));
  'update_metadata' => $auth->getUpdateMetadataFunc(),
];
$client = new helloworld\GreeterClient('greeter.googleapis.com', $opts);
```

#### Dart

##### 基础案例 - 无加密或认证 - Base case - no encryption or authentication

```dart
final channel = new ClientChannel('localhost',
      port: 50051,
      options: const ChannelOptions(
          credentials: const ChannelCredentials.insecure()));
final stub = new GreeterClient(channel);
```

##### 使用服务器认证的 SSL/TLS - With server authentication SSL/TLS

```dart
// Load a custom roots file.
// 加载自定义根证书文件。
final trustedRoot = new File('roots.pem').readAsBytesSync();
final channelCredentials =
    new ChannelCredentials.secure(certificates: trustedRoot);
final channelOptions = new ChannelOptions(credentials: channelCredentials);
final channel = new ClientChannel('myservice.example.com',
    options: channelOptions);
final client = new GreeterClient(channel);
```

##### 使用 Google 认证 Authenticate with Google

```dart
// Uses publicly trusted roots by default.
// 默认使用受信任的公开根证书。
final channel = new ClientChannel('greeter.googleapis.com');
final serviceAccountJson =
     new File('service-account.json').readAsStringSync();
final credentials = new JwtServiceAccountAuthenticator(serviceAccountJson);
final client =
    new GreeterClient(channel, options: credentials.toCallOptions);
```

##### 认证单个 RPC 调用 Authenticate a single RPC call

```dart
// Uses publicly trusted roots by default.
// 默认使用受信任的公开根证书。
final channel = new ClientChannel('greeter.googleapis.com');
final client = new GreeterClient(channel);
...
final serviceAccountJson =
     new File('service-account.json').readAsStringSync();
final credentials = new JwtServiceAccountAuthenticator(serviceAccountJson);
final response =
    await client.sayHello(request, options: credentials.toCallOptions);
```
