# iOS

{% hint style="success" %}
应的[官方页面地址](https://docs.passwordless.dev/guide/frontend/ios.html)
{% endhint %}

此 SDK 的目的是能够通过 Passwordless.dev 管理系统轻松地将通行密钥集成到您的 iOS App 中。

## apple-app-site-association

Apple 要求设置关联域名以执行通行密钥注册和断言。有关设置 apple-app-site-association 的更多详细信息，请参阅 [Apple 文档](https://developer.apple.com/documentation/Xcode/supporting-associated-domains)。

对于通行密钥，您需要为您的 App 设置一个 `webcredentials` 部分。

## iOS SDK 入门 <a href="#getting-started-with-the-ios-sdk" id="getting-started-with-the-ios-sdk"></a>

### 要求 <a href="#requirements" id="requirements"></a>

* **最低 iOS 版本：** iOS 16
* **额外功能：**
  * _关联域&#x540D;_&#x662F;添加 AASA 文件托管的域名所必需的。

### Swift 包管理器 <a href="#swift-package-manager" id="swift-package-manager"></a>

您可以通过 [Swift 包管理器](https://www.swift.org/package-manager/)将 SDK 添加到项目中。然后，您可以使用 `import Passwordless` 来访问 SDK 功能。

### CocoaPods

SDK 可以通过 CocoaPods 添加到您的项目中。在 Podfile 中指定以下内容：

```
pod 'Passwordless', '~> 1.0.0'
```

### 初始化 <a href="#initialization" id="initialization"></a>

SDK 提供了一个 `PasswordlessClient` 对象，该对象需要使用一个 `PasswordlessConfig` 配置进行初始化。配置包含与 [Passwordless.dev](https://bitwarden.com/products/passwordless/) 交互所需的 API URL 和 API 密钥。它还需要依赖方 ID，该 ID 代表您服务器上的域名，该域名托管了一个包含您的 App 团队 ID 和包 ID 的 apple-app-site-association 文件。

您需要注册 [Passwordless.dev](https://bitwarden.com/products/passwordless/) 以获取 API 密钥。

下面是一个配置示例：

```swift
let passwordlessClient = PasswordlessClient(
    config: PasswordlessConfig(
        apiKey: "<YOUR API KEY>",
        rpId: "demo.passwordless.dev"
    )
)
```

### 注册 <a href="#registration" id="registration"></a>

1. 使用给定的用户名从您的公共 RESTful API 请求注册令牌。
2. 通过 `register(token:)` 将注册令牌传递给 `PasswordlessClient` 。将提示用户使用生物识别创建所需的本地私钥以进行验证，并将公钥发送到 Passwordless.dev API。完成后，将返回一个验证令牌。请参阅[错误部分](ios.md#error-responses)以了解可能抛出的错误。

```swift
let verifyToken = try await passwordlessClient.register(token: registrationToken)
```

3. 将验证令牌传递给您的公共 RESTful API 以进行验证并返回一个授权令牌。用户现在已登录到您的 App 了。

![Registration Flow](https://docs.passwordless.dev/assets/Registration-Daec0Jbd.gif)

### 登录 <a href="#sign-in" id="sign-in"></a>

通行密钥登录主要有两种方式：自动填充和别名。同时也提供了用于更高级用例的 `signinWithDiscoverable()` 和 `signIn(userId:)`。

#### A. 自动填充 <a href="#a-auto-fill" id="a-auto-fill"></a>

当用户点击用户名字段时，键盘将出现。如果用户点击键盘上方的自动填充选项，那么这就是自动填充方式。为了正常工作，您必须在视图首次出现时调用 `signInWithAutofill`。这样，当键盘出现时，操作系统将准备好要显示的结果。

1. 在视图首次出现时调用 `signInWithAutofill()`。完成后，将返回一个验证令牌。
   * 此函数将等待用户点击自动填充项目、在通行密钥对话框中取消或发生错误。
   * 如果抛出错误，您的 App 需要决定是否重新启动自动填充登录过程。
   * 如果它没有运行，则键盘将不会显示任何选项，因此通常只有在 `授权已取消` 的情况下才需要重新启动该过程。
   * 获取 `授权错误` 可能表明您的 App 中某些配置不正确，因此最好不要再次运行自动填充以防止无限循环错误。请参阅[错误部分](ios.md#error-responses)以了解可能抛出的错误。

```swift
let verifyToken = try await passwordlessClient.signInWithAutofill()
```

2. 将验证令牌传递给您的公共 RESTful API 以进行验证并返回一个授权令牌。用户现在已登录到您的 App 了。

![Auto Fill Sign in Flow](https://docs.passwordless.dev/assets/SignInAutoFill-DbBxJjcH.gif)

#### B. 别名 <a href="#b-alias" id="b-alias"></a>

当用户输入用户名并点击视图中的按钮进行登录时，将显示供用户选择的不同的通行密钥窗口版本。

1. 使用给定的别名调用 `signIn(alias:)`。完成后，将返回一个验证令牌。请参阅[错误部分](ios.md#error-responses)以了解可能抛出的错误。

```
let verifyToken = try await passwordlessClient.signIn(alias: username)
```

2. 将验证令牌传递给您的公共 RESTful API 以进行验证并返回一个授权令牌。用户现在已登录到您的 App 了。

![Alias Sign in Flow](https://docs.passwordless.dev/assets/SignInManual-Ta_JSAtv.gif)

#### C. 可被发现 <a href="#c-discoverable" id="c-discoverable"></a>

如果您想手动显示通行密钥模态，而不使用快捷键或让用户输入别名。

#### D. 用户 ID <a href="#d-user-id" id="d-user-id"></a>

如果您已经有了用户 ID，并想进行断言过程。

### 错误响应 <a href="#error-responses" id="error-responses"></a>

PasswordlessClient 对象可能会抛出以下错误：

| 错误                                                                         | 描述                         |
| -------------------------------------------------------------------------- | -------------------------- |
| authorizationCancelled                                                     | 操作系统授权被密钥凭证提供程序取消。         |
| authorizationError(Error)                                                  | 来自密钥凭证提供程序的操作系统授权失败。       |
| internalErrorDecodingJson(Error)                                           | 从网络响应中解码 JSON 到对象时出现问题。    |
| internalErrorEncodingPayload(Error)                                        | 从对象编码 JSON 以进行网络请求时出现问题。   |
| internalErrorInvalidURL(String)                                            | 使用的网络请求 URL 无效。            |
| internalErrorNetworkRequestFailed(Error)                                   | 在进行网络请求时发生错误。              |
| internalErrorNetworkRequestResponseError(Int?, PasswordlessErrorResponse?) | 网络请求发生错误，如果可用，将提供状态码和错误响应。 |
| internalErrorUnableToDecodeChallenge                                       | 提供的挑战格式不正确。                |
| internalErrorUnableToEncodeUserId                                          | 用户 ID 无法被编码为 Base64 URL。   |

对于 internalErrorNetworkRequestResponseError，可以提供 PasswordlessErrorResponse，其中包含关于响应中错误的详细信息。以下是一个可能返回的响应错误示例：

```json
{
  "type": "https://docs.passwordless.dev/errors#missing_register_token",
  "title": "The token you sent was not correct. The token used for this endpoint should start with 'register_'. Make sure you are not sending the wrong value.",
  "status": 400,
  "errorCode": "missing_register_token"
}
```

下面是一个如何在您的 App 中查看此示例的示例：

```swift
do {
    let verifyToken = try await passwordlessClient.signIn()
}
catch PasswordlessClientError.authorizationCancelled {
    print("Cancelled")
}
catch let PasswordlessClientError.internalErrorNetworkRequestResponseError(code, response) {
    print("Network Error occurred. Response status code: \(code ?? -1)")
    dump(response)
}
catch {
    print("Some other error occurred: \(error)")
}
```

