+++
title = "Authentication"
date = 2024-01-17T08:51:13+08:00
weight = 1
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文: [https://grpc.io/docs/guides/auth/](https://grpc.io/docs/guides/auth/)

# Authentication 认证

An overview of gRPC authentication, including built-in auth mechanisms, and how to plug in your own authentication systems.

​	gRPC 身份验证概述，包括内置身份验证机制，以及如何插入您自己的身份验证系统。



### Overview 概述

gRPC is designed to work with a variety of authentication mechanisms, making it easy to safely use gRPC to talk to other systems. You can use our supported mechanisms - SSL/TLS with or without Google token-based authentication - or you can plug in your own authentication system by extending our provided code.

​	gRPC 旨在与各种身份验证机制配合使用，从而轻松安全地使用 gRPC 与其他系统通信。您可以使用我们支持的机制 - 带有或不带有基于 Google 令牌的身份验证的 SSL/TLS - 或者通过扩展我们提供的代码来插入您自己的身份验证系统。

gRPC also provides a simple authentication API that lets you provide all the necessary authentication information as `Credentials` when creating a channel or making a call.

​	gRPC 还提供了一个简单的身份验证 API，让您可以在创建频道或拨打电话时提供所有必要的身份验证信息作为 `Credentials` 。

### Supported auth mechanisms 支持的身份验证机制

The following authentication mechanisms are built-in to gRPC:

​	以下身份验证机制内置于 gRPC 中：

- **SSL/TLS**: gRPC has SSL/TLS integration and promotes the use of SSL/TLS to authenticate the server, and to encrypt all the data exchanged between the client and the server. Optional mechanisms are available for clients to provide certificates for mutual authentication.
  SSL/TLS：gRPC 具有 SSL/TLS 集成，并促进使用 SSL/TLS 来对服务器进行身份验证，并加密客户端与服务器之间交换的所有数据。可选机制可供客户端提供证书以进行相互身份验证。
- **ALTS**: gRPC supports [ALTS](https://cloud.google.com/security/encryption-in-transit/application-layer-transport-security) as a transport security mechanism, if the application is running on [Compute Engine](https://cloud.google.com/compute) or [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine). For details, see one of the following language-specific pages: [ALTS in C++]({{< ref "/Languages/C/ALTS" >}}), [ALTS in Go]({{< ref "/Languages/Go/ALTS" >}}), [ALTS in Java]({{< ref "/Languages/Java/ALTS" >}}), [ALTS in Python]({{< ref "/Languages/Python/ALTS" >}}).
  ALTS：如果应用程序在 Compute Engine 或 Google Kubernetes Engine (GKE) 上运行，gRPC 将 ALTS 作为传输安全机制提供支持。有关详细信息，请参阅以下语言特定页面之一：C++ 中的 ALTS、Go 中的 ALTS、Java 中的 ALTS、Python 中的 ALTS。
- **Token-based authentication with Google**: gRPC provides a generic mechanism (described below) to attach metadata based credentials to requests and responses. Additional support for acquiring access tokens (typically OAuth2 tokens) while accessing Google APIs through gRPC is provided for certain auth flows: you can see how this works in our code examples below. In general this mechanism must be used *as well as* SSL/TLS on the channel - Google will not allow connections without SSL/TLS, and most gRPC language implementations will not let you send credentials on an unencrypted channel.
  基于令牌的 Google 身份验证：gRPC 提供了一种通用机制（如下所述）来将基于元数据的凭据附加到请求和响应。在通过 gRPC 访问 Google API 时，还提供了获取访问令牌（通常是 OAuth2 令牌）的其他支持，以用于某些身份验证流程：您可以在下面的代码示例中看到它的工作原理。通常，此机制必须与通道上的 SSL/TLS 一起使用 - Google 不允许没有 SSL/TLS 的连接，并且大多数 gRPC 语言实现不允许您在未加密的通道上发送凭据。

#### Warning 警告

Google credentials should only be used to connect to Google services. Sending a Google issued OAuth2 token to a non-Google service could result in this token being stolen and used to impersonate the client to Google services.
Google 凭据只能用于连接到 Google 服务。将 Google 颁发的 OAuth2 令牌发送到非 Google 服务可能会导致此令牌被盗并用于冒充客户端访问 Google 服务。

### Authentication API 身份验证 API

gRPC provides a simple authentication API based around the unified concept of Credentials objects, which can be used when creating an entire gRPC channel or an individual call.

​	gRPC 提供了一个简单的身份验证 API，该 API 基于凭据对象的统一概念，可在创建整个 gRPC 通道或单个调用时使用。

#### Credential types 凭据类型

Credentials can be of two types:

​	凭据可以是两种类型：

- **Channel credentials**, which are attached to a `Channel`, such as SSL credentials.
  通道凭据，附加到 `Channel` ，例如 SSL 凭据。
- **Call credentials**, which are attached to a call (or `ClientContext` in C++).
  调用凭据，附加到调用（或在 C++ 中为 `ClientContext` ）。

You can also combine these in a `CompositeChannelCredentials`, allowing you to specify, for example, SSL details for the channel along with call credentials for each call made on the channel. A `CompositeChannelCredentials` associates a `ChannelCredentials` and a `CallCredentials` to create a new `ChannelCredentials`. The result will send the authentication data associated with the composed `CallCredentials` with every call made on the channel.

​	您还可以在 `CompositeChannelCredentials` 中组合这些凭据，这允许您指定例如通道的 SSL 详细信息以及通道上进行的每个调用的调用凭据。 `CompositeChannelCredentials` 将 `ChannelCredentials` 和 `CallCredentials` 关联起来以创建新的 `ChannelCredentials` 。结果将发送与组合的 `CallCredentials` 关联的身份验证数据，用于在通道上进行的每次调用。

For example, you could create a `ChannelCredentials` from an `SslCredentials` and an `AccessTokenCredentials`. The result when applied to a `Channel` would send the appropriate access token for each call on this channel.

​	例如，您可以从 `SslCredentials` 和 `AccessTokenCredentials` 创建 `ChannelCredentials` 。当应用于 `Channel` 时，结果将为该通道上的每个调用发送适当的访问令牌。

Individual `CallCredentials` can also be composed using `CompositeCallCredentials`. The resulting `CallCredentials` when used in a call will trigger the sending of the authentication data associated with the two `CallCredentials`.

​	还可以使用 `CompositeCallCredentials` 组合各个 `CallCredentials` 。当在调用中使用生成的 `CallCredentials` 时，将触发发送与两个 `CallCredentials` 关联的身份验证数据。

#### Using client-side SSL/TLS 使用客户端 SSL/TLS

Now let’s look at how `Credentials` work with one of our supported auth mechanisms. This is the simplest authentication scenario, where a client just wants to authenticate the server and encrypt all data. The example is in C++, but the API is similar for all languages: you can see how to enable SSL/TLS in more languages in our Examples section below.

​	现在，我们来看看 `Credentials` 如何与我们支持的一种身份验证机制配合使用。这是最简单的身份验证方案，其中客户端只想对服务器进行身份验证并加密所有数据。示例使用 C++，但 API 对所有语言都类似：您可以在下面的示例部分中了解如何在更多语言中启用 SSL/TLS。

```cpp
// Create a default SSL ChannelCredentials object.
auto channel_creds = grpc::SslCredentials(grpc::SslCredentialsOptions());
// Create a channel using the credentials created in the previous step.
auto channel = grpc::CreateChannel(server_name, channel_creds);
// Create a stub on the channel.
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
// Make actual RPC calls on the stub.
grpc::Status s = stub->sayHello(&context, *request, response);
```

For advanced use cases such as modifying the root CA or using client certs, the corresponding options can be set in the `SslCredentialsOptions` parameter passed to the factory method.

​	对于高级用例，例如修改根 CA 或使用客户端证书，可以在传递给工厂方法的 `SslCredentialsOptions` 参数中设置相应的选项。

#### Note 注意

Non-POSIX-compliant systems (such as Windows) need to specify the root certificates in `SslCredentialsOptions`, since the defaults are only configured for POSIX filesystems.
不兼容 POSIX 的系统（例如 Windows）需要在 `SslCredentialsOptions` 中指定根证书，因为默认值仅针对 POSIX 文件系统配置。

#### Using OAuth token-based authentication 使用基于 OAuth 令牌的身份验证

OAuth 2.0 Protocol is the industry-standard protocol for authorization. It enables websites or applications to obtain limited access to user accounts using OAuth tokens.

​	OAuth 2.0 协议是用于授权的行业标准协议。它使网站或应用程序能够使用 OAuth 令牌获取对用户帐户的有限访问权限。

gRPC offers a set of simple APIs to integrate OAuth 2.0 into applications, streamlining authentication.

​	gRPC 提供了一组简单的 API，可将 OAuth 2.0 集成到应用程序中，从而简化身份验证。

At a high level, using OAuth token-based authentication includes 3 steps:

​	从高层次来看，使用基于 OAuth 令牌的身份验证包括 3 个步骤：

1. Get or generate an OAuth token on client side.

   
   在客户端获取或生成 OAuth 令牌。

   - You can generate Google-specific tokens following instructions below.
     您可以按照以下说明生成特定于 Google 的令牌。

2. Create credentials with the OAuth token.

   
   使用 OAuth 令牌创建凭据。

   - OAuth token is always part of per-call credentials, you can also attach the per-call credentials to some channel credentials.
     OAuth 令牌始终是按调用凭据的一部分，您还可以将按调用凭据附加到某些通道凭据。
   - The token will be sent to server, normally as part of HTTP Authorization header.
     令牌将发送到服务器，通常作为 HTTP 授权标头的一部分。

3. Server side verifies the token.

   
   服务器端验证令牌。

   - In most implementations, the validation is done using a server side interceptor.
     在大多数实现中，验证是使用服务器端拦截器完成的。

For details of how to use OAuth token in different languages, please refer to our examples below.

​	有关如何在不同语言中使用 OAuth 令牌的详细信息，请参阅以下示例。

#### Using Google token-based authentication 使用基于 Google 令牌的身份验证

gRPC applications can use a simple API to create a credential that works for authentication with Google in various deployment scenarios. Again, our example is in C++ but you can find examples in other languages in our Examples section.

​	gRPC 应用程序可以使用简单的 API 来创建凭据，该凭据适用于在各种部署场景中使用 Google 进行身份验证。同样，我们的示例使用 C++，但您可以在示例部分中找到其他语言的示例。

```cpp
auto creds = grpc::GoogleDefaultCredentials();
// Create a channel, stub and make RPC calls (same as in the previous example)
auto channel = grpc::CreateChannel(server_name, creds);
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
grpc::Status s = stub->sayHello(&context, *request, response);
```

This channel credentials object works for applications using Service Accounts as well as for applications running in [Google Compute Engine (GCE)](https://cloud.google.com/compute/). In the former case, the service account’s private keys are loaded from the file named in the environment variable `GOOGLE_APPLICATION_CREDENTIALS`. The keys are used to generate bearer tokens that are attached to each outgoing RPC on the corresponding channel.

​	此频道凭据对象适用于使用服务帐号的应用程序以及在 Google Compute Engine (GCE) 中运行的应用程序。在前一种情况下，服务帐号的私钥从环境变量 `GOOGLE_APPLICATION_CREDENTIALS` 中命名的文件中加载。这些密钥用于生成附带到相应频道上每个传出 RPC 的持有者令牌。

For applications running in GCE, a default service account and corresponding OAuth2 scopes can be configured during VM setup. At run-time, this credential handles communication with the authentication systems to obtain OAuth2 access tokens and attaches them to each outgoing RPC on the corresponding channel.

​	对于在 GCE 中运行的应用程序，可以在虚拟机设置期间配置默认服务帐号和相应的 OAuth2 范围。在运行时，此凭据处理与身份验证系统的通信以获取 OAuth2 访问令牌，并将它们附加到相应通道上的每个传出 RPC。

#### Extending gRPC to support other authentication mechanisms 扩展 gRPC 以支持其他身份验证机制

The Credentials plugin API allows developers to plug in their own type of credentials. This consists of:

​	凭据插件 API 允许开发者插入他们自己的凭据类型。这包括：

- The `MetadataCredentialsPlugin` abstract class, which contains the pure virtual `GetMetadata` method that needs to be implemented by a sub-class created by the developer.
  `MetadataCredentialsPlugin` 抽象类，其中包含纯虚拟 `GetMetadata` 方法，需要由开发者创建的子类实现。
- The `MetadataCredentialsFromPlugin` function, which creates a `CallCredentials` from the `MetadataCredentialsPlugin`.
  `MetadataCredentialsFromPlugin` 函数，它从 `MetadataCredentialsPlugin` 创建 `CallCredentials` 。

Here is example of a simple credentials plugin which sets an authentication ticket in a custom header.

​	这是一个简单凭据插件的示例，它在自定义标头中设置身份验证票证。

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

​	可以在核心级别插入 gRPC 凭据实现来实现更深入的集成。gRPC 内部还允许使用其他加密机制替换 SSL/TLS。

### Language guides and examples 语言指南和示例

These authentication mechanisms will be available in all gRPC’s supported languages. The following table links to examples demonstrating authentication and authorization in various languages.

​	这些身份验证机制将在 gRPC 支持的所有语言中提供。下表链接到演示各种语言中的身份验证和授权的示例。

| Language 语言 | Example 示例                                                 | Documentation 文档                                           |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| C++           | N/A                                                          | N/A                                                          |
| Go            | [Go Example Go 示例](https://github.com/grpc/grpc-go/tree/master/examples/features/encryption) | [Go Documentation Go 文档](https://github.com/grpc/grpc-go/tree/master/examples/features/encryption#encryption) |
| Java          | [Java Example TLS](https://github.com/grpc/grpc-java/tree/master/examples/example-tls) ([Java Example ATLS](https://github.com/grpc/grpc-java/tree/master/examples/example-alts)) Java 示例 TLS（Java 示例 ATLS） | [Java Documentation Java 文档](https://github.com/grpc/grpc-java/tree/master/examples/example-tls#hello-world-example-with-tls) |
| Python        | [Python Example Python 示例](https://github.com/grpc/grpc/tree/master/examples/python/auth) | [Python Documentation Python 文档](https://github.com/grpc/grpc/tree/master/examples/python/auth#authentication-extension-example-in-grpc-python) |

### Language guides and examples for OAuth token-based authentication OAuth 令牌身份验证的语言指南和示例

The following table links to examples demonstrating OAuth token-based authentication and authorization in various languages.

​	下表链接到演示各种语言中基于 OAuth 令牌的身份验证和授权的示例。

| Language 语言 | Example 示例                                                 | Documentation 文档                                           |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| C++           | N/A                                                          | N/A                                                          |
| Go            | [Go OAuth Example Go OAuth 示例](https://github.com/grpc/grpc-go/tree/master/examples/features/authentication#authentication) | [Go OAuth Documentation Go OAuth 文档](https://github.com/grpc/grpc-go/tree/master/examples/features/authentication#oauth2) |
| Java          | [Java OAuth Example Java OAuth 示例](https://github.com/grpc/grpc-java/tree/master/examples/example-oauth#authentication-example) | [Java OAuth Documentation Java OAuth 文档](https://github.com/grpc/grpc-java/tree/master/examples/example-oauth) |
| Python        | [Python OAuth Example Python OAuth 示例](https://github.com/grpc/grpc/blob/master/examples/python/auth/token_based_auth_client.py) | [Python OAuth Documentation Python OAuth 文档](https://github.com/grpc/grpc/tree/master/examples/python/auth#token-based-authentication) |

### Additional Examples 其他示例

The following sections demonstrate how authentication and authorization features described above appear in other languages not listed above.

​	以下部分演示了上面描述的身份验证和授权功能如何出现在上面未列出的其他语言中。

#### Ruby

##### Base case - no encryption or authentication 基本情况 - 无加密或身份验证

```ruby
stub = Helloworld::Greeter::Stub.new('localhost:50051', :this_channel_is_insecure)
...
```

##### With server authentication SSL/TLS 使用服务器身份验证 SSL/TLS

```ruby
creds = GRPC::Core::ChannelCredentials.new(load_certs)  # load_certs typically loads a CA roots file
stub = Helloworld::Greeter::Stub.new('myservice.example.com', creds)
```

##### Authenticate with Google 使用 Google 身份验证

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

##### Base case - No encryption/authentication 基本情况 - 无加密/身份验证

```js
var stub = new helloworld.Greeter('localhost:50051', grpc.credentials.createInsecure());
```

##### With server authentication SSL/TLS 使用服务器身份验证 SSL/TLS

```js
const root_cert = fs.readFileSync('path/to/root-cert');
const ssl_creds = grpc.credentials.createSsl(root_cert);
const stub = new helloworld.Greeter('myservice.example.com', ssl_creds);
```

##### Authenticate with Google 使用 Google 身份验证

```js
// Authenticating with Google
var GoogleAuth = require('google-auth-library'); // from https://www.npmjs.com/package/google-auth-library
...
var ssl_creds = grpc.credentials.createSsl(root_certs);
(new GoogleAuth()).getApplicationDefault(function(err, auth) {
  var call_creds = grpc.credentials.createFromGoogleCredential(auth);
  var combined_creds = grpc.credentials.combineChannelCredentials(ssl_creds, call_creds);
  var stub = new helloworld.Greeter('greeter.googleapis.com', combined_credentials);
});
```

##### Authenticate with Google using OAuth2 token (legacy approach) 使用 OAuth2 令牌对 Google 进行身份验证（旧方法）

```js
var GoogleAuth = require('google-auth-library'); // from https://www.npmjs.com/package/google-auth-library
...
var ssl_creds = grpc.Credentials.createSsl(root_certs); // load_certs typically loads a CA roots file
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

##### With server authentication SSL/TLS and a custom header with token 使用服务器身份验证 SSL/TLS 和带有令牌的自定义标头

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

##### Base case - No encryption/authorization 基本情况 - 无加密/授权

```php
$client = new helloworld\GreeterClient('localhost:50051', [
    'credentials' => Grpc\ChannelCredentials::createInsecure(),
]);
```

##### With server authentication SSL/TLS 使用服务器身份验证 SSL/TLS

```php
$client = new helloworld\GreeterClient('myservice.example.com', [
    'credentials' => Grpc\ChannelCredentials::createSsl(file_get_contents('roots.pem')),
]);
```

##### Authenticate with Google 使用 Google 进行身份验证

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

##### Authenticate with Google using OAuth2 token (legacy approach) 使用 OAuth2 令牌对 Google 进行身份验证（旧方法）

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

##### Base case - no encryption or authentication 基本情况 - 无加密或身份验证

```dart
final channel = new ClientChannel('localhost',
      port: 50051,
      options: const ChannelOptions(
          credentials: const ChannelCredentials.insecure()));
final stub = new GreeterClient(channel);
```

##### With server authentication SSL/TLS 使用服务器身份验证 SSL/TLS

```dart
// Load a custom roots file.
final trustedRoot = new File('roots.pem').readAsBytesSync();
final channelCredentials =
    new ChannelCredentials.secure(certificates: trustedRoot);
final channelOptions = new ChannelOptions(credentials: channelCredentials);
final channel = new ClientChannel('myservice.example.com',
    options: channelOptions);
final client = new GreeterClient(channel);
```

##### Authenticate with Google 使用 Google 进行身份验证

```dart
// Uses publicly trusted roots by default.
final channel = new ClientChannel('greeter.googleapis.com');
final serviceAccountJson =
     new File('service-account.json').readAsStringSync();
final credentials = new JwtServiceAccountAuthenticator(serviceAccountJson);
final client =
    new GreeterClient(channel, options: credentials.toCallOptions);
```

##### Authenticate a single RPC call 对单个 RPC 调用进行身份验证

```dart
// Uses publicly trusted roots by default.
final channel = new ClientChannel('greeter.googleapis.com');
final client = new GreeterClient(channel);
...
final serviceAccountJson =
     new File('service-account.json').readAsStringSync();
final credentials = new JwtServiceAccountAuthenticator(serviceAccountJson);
final response =
    await client.sayHello(request, options: credentials.toCallOptions);
```
