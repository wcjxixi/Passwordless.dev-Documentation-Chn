# 概念

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/concepts.html)
{% endhint %}

> \[**译者注**]：[CATP](https://en.wikipedia.org/wiki/Client_to_Authenticator_Protocol)：Client to Authenticator Protocol（客户端到验证器协议）

## FIDO2 <a href="#fido2" id="fido2"></a>

FIDO2 是万维网联盟 (W3C) 关于网络身份验证 (WebAuthn) 和客户端到验证器协议 (CTAP) 的标准规范。制定 FIDO 身份验证标准是为了提供比标准密码和 SMS 2FA 更安全的身份验证。使用 FIDO 身份验证标准可以提供一种安全的体验，让消费者更容易使用，让开发人员更容易实施。有关 FIDO 的更多信息，请访问 [FIDO Alliance](https://fidoalliance.org/fido2/)。

FIDO2 由 **WebAuthn** 和 **CTAP** 两个标准化组件组成。这些标准的共同作用是创造一种安全的无密码体验。

* **WebAuthn** - 是一种应用程序接口 (API)，可将依赖方连接到应用程序或登录系统。从实际意义上讲，WebAuthn 在网络和应用程序之间建立了一种简便的连接，允许进行无密码身份验证。点击[此处](https://www.yubico.com/resource/why-webauthn-matters/)了解有关 WebAuthn 的更多信息。
* **CTAP2** - 客户端到验证器协议组件允许外部和便携式验证器（安全钥匙）与客户端平台一起运行。FIDO CTAP2 负责外部因素（如安全钥匙），通过验证器与网站或账户进行通信。

为了符合 FIDO2 标准，Passwordless.dev 身份验证过程将同时采用 WebAuthn 和 CTAP2 标准。

### 通行密钥 <a href="#passkeys" id="passkeys"></a>

通行密钥 (Passkey) 是密码的替代品，可以让用户在不同设备上更快、更方便、更安全地登录网站和应用程序。更确切地说，「通行密匙」是一个对消费者友好的术语，指的是一种可被发现的 FIDO 凭证，它可以通过同步实现跨设备的安全无密码登录，也可以作为设备绑定的通行密钥专用于单个硬件。

当与设备绑定时，通行密钥还可以提供升级的认证功能。

### 可发现凭据 <a href="#discoverable-credential" id="discoverable-credential"></a>

可发现凭证是可用于身份验证的通行密钥，服务器不需要先输入 `credentialId`。这意味着您无需首先使用用户名或电子邮件来识别用户，从而使登录变得更加简单。通行密钥是可发现凭证的一个示例。

### 用户标识符 <a href="#user-identifiers" id="user-identifiers"></a>

FIDO2 规范定义了多个用户标识符，Passwordless.dev 可以在各种注册和登录操作中使用这些标识符：

* **userId** - 代表 [WebAuthn 用户句柄](https://www.w3.org/TR/webauthn-2/#dom-publickeycredentialuserentity-id)的唯一字符串。该值不会显示给用户，也不应包含个人身份信息。身份验证尝试只能根据 `userId` 值进行。`userId` 的一个示例是数据库主键，例如 int（整数） `123` 或 guid `a2bd8bf7-2145-4a5a-910f-8fdc9ef421d3`。
* **用户名** - （仅用于显示目的）是用户账户的人性化标识符。它仅用于显示，即帮助用户区分显示名称相似的用户账户。在浏览器 UI 中使用，从不存储在数据库中。例如 `pjfry@passwordless.dev`
* **显示名称** - （仅用于显示目的）是账户的人性化名称，应由用户选择，仅在应用程序的 UI 中使用。例如 `Philip J. Fry`
* **别名** - 是面向用户的对 `userId` 的引用，允许使用其他用户名、电子邮件地址等登录。默认情况下，别名在存储之前会进行哈希处理，以保护用户隐私。可以通过向 `/alias` 端点发出请求来为 `userId` 设置多个别名（[了解更多](api.md#alias)），但是在允许用户创建别名时应考虑以下规则：
  * 别名对于指定的 `userId` 必须是唯一的。
  * 别名长度不得超过 250 个字符。
  * 一个 `userId` 最多可以有 10 个与其关联的别名。

### 身份验证器类型 <a href="#authenticator-types" id="authenticator-types"></a>

FIDO2 身份验证器有两种类型：

* **平台身份验证器**是设备驻留的身份验证器，例如 macOS 的 FaceID 或 TouchID、或 Windows Hello，无法通过 USB 或 NFC 等协议进行访问。
* **漫游身份验证器**（也称为「跨平台」）是可拆卸的、与设备无关的身份验证器，例如 USB 安全钥匙，可通过 USB 或 NFC 等支持的传输协议连接到多个设备。

## Passwordless.dev <a href="#passwordless-dev" id="passwordless-dev"></a>

### 产品组件 <a href="#product-components" id="product-components"></a>

从架构上来说，Passwordless.dev 由三个关键部分组成：

* 一个[开源的客户端库](frontend/javascript.md)，由您的前端使用它来向最终用户浏览器的 WebAuthn API 发出请求以及向 Passwordless.dev API 发出请求。
* 一个公共 RESTful API，由您的前端使用它来完成与浏览器的 FIDO2 WebAuthn 加密交换。
* 一个[私有 RESTful API](api.md)，由您的后端使用它来发起密钥注册、验证登录以及为最终用户检索密钥。

### API 密钥 <a href="#api-keys" id="api-keys"></a>

使用 [Passwordless.dev 管理控制台](get-started.md#create-an-application)注册应用程序将创建一组 API 密钥：

*   **ApiKey**：一种公共 API 密钥，安全且被包含在客户端侧。它允许浏览器连接到我们的后端并发起密钥协商和断言。公共 API 密钥的格式为：

    ```
    <application-name>:public:<guid>
    //e.g. myapp:public:a28e285ec8b64ca58a3dec90c5af48c2
    ```
*   **ApiSecret**：一种私有 API 密钥或私有机密，应受到妥善保护。它允许您的后端代表您的用户验证登录和注册密钥。私有 API 机密的格式为：

    ```
    <application-name>:secret:<guid>
    //e.g. myapp:secret:42cd551fb288371812596e211fbc2a5a
    ```

### 凭据 <a href="#credential" id="credential"></a>

凭据代表了由 Passwordless.dev 为用户注册的 FIDO2 身份验证器。凭凭据示例包括通行密钥和硬件安全钥匙。每个凭据都存储了以下信息：

| 属性                 | 描述                                                 |
| ------------------ | -------------------------------------------------- |
| `descriptorId`     | 标识特定凭据的字节数组的 Base64Url 字符串表示形式。也称为 `credentialId`. |
| `publicKey`        | 凭证的公钥，用于以加密方式验证身份验证。注意：知道公钥**并不**意味着可以访问帐账户/凭据。    |
| `userId`           | 与特定用户账户关联的唯一标识符。它可用于检索有关用户的信息。例如 `123`。            |
| `signatureCounter` | 使用此凭证进行身份验证的使用次数。                                  |
| `createdAt`        | 为应用程序注册凭证的时间戳 (UTC)。                               |
| `aaGuid`           | 身份验证器验证 GUID 是一个唯一标识符，用于在注册身份验证器时识别它。              |
| `lastUsedAt`       | 证书最后一次用于应用程序身份验证的时间戳 (UTC)。                        |
| `rpid`             | 为应用程序注册凭证的依赖方标识符。                                  |
| `origin`           | 使用 API 的服务的域名或 IP 地址。                              |
| `country`          | 表示凭证所在地或注册地的国家代码。                                  |
| `device`           | 全权证书所在设备的设备信息，例如 `Chrome、Mac OS X 10`。             |
| `nickname`         | 用户指定的与此特定凭据相关联的名称，例如 `My Macbook`。                 |

### 令牌 <a href="#tokens" id="tokens"></a>

在正常业务过程中，Passwordless.dev 使用两种重要类型的临时令牌：

* **注册令牌**，由私有 API 通过请求 `/register/token` 端点创建（[了解更多](api.md#register-token)）。您的前端将在最终用户的设备上注册此令牌，以便在登录操作中使用。
* **身份验证令牌**，由公共 API 通过调用 `.signin()` 方法创建（[了解更多](api.md#signin-verify)）。您的后端将验证此令牌以完成登录操作（通过 `/signin/verify` 端点）。

此外，Passwordless.dev 还使用其他类型的令牌用于特殊目的：

* **手动生成的身份验证令牌**，由私有 API 通过对 `/signin/generate-token` 端点的请求创建。该令牌的权重与常规身份验证令牌相同，但它是手动生成的，绕过了常规身份验证流程。它主要用于方便账户恢复和通过魔法链接登录。

### Attestation

{% hint style="warning" %}
仅我们的「企业」计划中支持 Attestation。[了解更多](https://bitwarden.com/products/passwordless/#pricing)有关我们的计划的信息。
{% endhint %}

虽然 WebAuthn 非常安全，但 Attestation 通过允许依赖方接收和验证有关验证器的信息，从而增强了 WebAuthn 注册过程的安全性。

| 类型           | 计划  | 描述                                                                                                                      |
| ------------ | --- | ----------------------------------------------------------------------------------------------------------------------- |
| `none`       | 所有  | 验证器提供其 Attestation 信息，但没有有意义的加密签名。与其它 Attestation 类型相比，安全性较低，通常在依赖方不需要强有力地证明验证器特征时使用。                                   |
| `direct`     | 企业  | 验证器直接向依赖方提供其 Attestation 信息。然而，需要注意的是，「直接」Attestation 实际上等同于「自我 Attestation」。验证器对其自身属性做出声明，而不涉及第三方 Attestation 凭据。      |
| `indirect`   | 企业  | 验证器包含来自 Attestation Certificate Authority (AttCA) 的签名声明。来自 AttCA 的 Attestation 声明提供了验证器的特征的加密证明，增强了 Attestation 过程的安全性。 |
| `enterprise` | n/a | 为企业提供了唯一识别验证器的功能，以确定它们是否被批准在受保护环境中使用。                                                                                   |

依赖方可以使用 Attestation 信息做出是否信任验证器的明智决定，并评估用户设备提供的安全级别。值得注意的是，虽然 Attestation 能增强安全性，但它并不是 WebAuthn 基本操作的强制性要求。在没有 Attestation 的情况下，WebAuthn 仍可正常工作，但建议使用 Attestation 来增强安全性。

### Attestation 配置 <a href="#authentication-configurations" id="authentication-configurations"></a>

Attestation 配置允许您配置在 `signin()` 和 `stepup()` 客户端方法中使用的认证令牌。每个方法都将参数传递给浏览器访问的认证器。Attestation 配置允许为给定的认证工作流程设置认证令牌的生存时间和用户验证要求设置。还有其他可用的配置选项。

每个应用程序都有两种默认的 Attestation 配置，分别是 `step-up` 和 `sign-in`。它们分别作为认证的 `purpose` 在各自的客户端方法中使用。它们可以被编辑，如果被删除，将恢复到默认设置。Attestation 配置可以通过 [API](api.md#auth-configs) 或管理控制台访问。

### 凭据提示 <a href="#credential-hints" id="credential-hints"></a>

在执行登录操作时，Passwordless.dev API 可以为用户代理提供如何最好地认证用户的提示。这些提示不是强制性的，用户代理有权选择如何响应它们。

以下是一些可用的提示：

* `SecurityKey` - 用户代理应使用安全密钥进行身份验证。
* `ClientDevice` - 用户代理应使用设备的内置验证器进行身份验证。
* `Hybrid` - 用户代理应使用通用验证器进行身份验证，例如智能手机。

凭证提示可以按顺序组合成一个列表，以向用户代理提供用户身份验证的偏好。用户代理应使用列表中它能满足的第一个提示。

在 Passwordless.dev 中，凭证提示是作为身份验证配置的一部分进行配置的。

## 更多术语 <a href="#more-terms" id="more-terms"></a>

### 依赖方 <a href="#relying-party" id="relying-party"></a>

依赖方 (RP) 是处理资源访问请求的服务器。网络应用程序在访问请求期间验证用户凭据就是一个 RP 的例子。

### 依赖方 ID <a href="#relying-party-id" id="relying-party-id"></a>

依赖方 ID 提供与给定域相对应的技术平台和标识。

### 用户验证 <a href="#user-verification" id="user-verification"></a>

FIDO2 服务器 RP（依赖方）与身份验证器交互，以验证用户。这可以通过 PIN 码、生物识别或其他 2FA 方法来完成，这些方法可以安全地验证访问账户的人是否正确。
