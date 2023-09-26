# 概念

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/concepts.html)
{% endhint %}

## FIDO2 <a href="#fido2" id="fido2"></a>

FIDO2 是万维网联盟 (W3C) 关于网络身份验证 (WebAuthn) 和客户端到验证器协议 (CTAP) 的标准规范。制定 FIDO 身份验证标准是为了提供比标准密码和 SMS 2FA 更安全的身份验证。使用 FIDO 身份验证标准可以提供一种安全的体验，让消费者更容易使用，让开发人员更容易实施。有关 FIDO 的更多信息，请访问 [FIDO Alliance](https://fidoalliance.org/fido2/)。

FIDO2 由 **WebAuthn** 和 **CTAP** 两个标准化组件组成。这些标准的共同作用是创造一种安全的无密码体验。

* **WebAuthn** - 是一种应用程序接口 (API)，可将依赖方连接到应用程序或登录系统。从实际意义上讲，WebAuthn 在网络和应用程序之间建立了一种简便的连接，允许进行无密码身份验证。点击[此处](https://www.yubico.com/resource/why-webauthn-matters/)了解有关 WebAuthn 的更多信息。
* **CTAP2** - 客户端到验证器协议组件允许外部和便携式验证器（安全密钥）与客户端平台一起运行。FIDO CTAP2 负责外部因素（如安全密钥），通过验证器与网站或账户进行通信。

为了符合 FIDO2 标准，Passwordless.dev 身份验证过程将同时采用 WebAuthn 和 CTAP2 标准。

### 通行密钥 <a href="#passkeys" id="passkeys"></a>

### 可发现凭据 <a href="#discoverable-credential" id="discoverable-credential"></a>

### 用户标识符 <a href="#user-identifiers" id="user-identifiers"></a>

### 身份验证器类型 <a href="#authenticator-types" id="authenticator-types"></a>

## Passwordless.dev <a href="#passwordless-dev" id="passwordless-dev"></a>

### 产品组件 <a href="#product-components" id="product-components"></a>

### API 密钥 <a href="#api-keys" id="api-keys"></a>

### 凭据 <a href="#credential" id="credential"></a>

### 令牌 <a href="#tokens" id="tokens"></a>

## 更多条款 <a href="#more-terms" id="more-terms"></a>

### 依赖方 <a href="#relying-party" id="relying-party"></a>

### 依赖方 ID <a href="#relying-party-id" id="relying-party-id"></a>

### 用户验证 <a href="#user-verification" id="user-verification"></a>
