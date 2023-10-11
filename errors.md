# 错误

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/errors.html)
{% endhint %}

虽然错误从来都不是一件有趣的事情，但我们还是尽力让您能轻松判断出错原因。以下是您可能遇到的各种错误以及解决方法。

## 问题信息详细 <a href="#problem-details" id="problem-details"></a>

所有错误都会按照 [RFC-7807](https://www.rfc-editor.org/rfc/rfc7807) 被格式化为问题详情，这意味着我们的 HTTP API 和客户端代码会返回类似这样的错误：

```json5
{
  "type": "https://docs.passwordless.dev/errors#missing_register_token",
  "title": "The token you sent was not correct. The token used for this endpoint should start with 'register_'. Make sure you are not sending the wrong value.",
  "status": 400,
  "errorCode": "missing_register_token"
}
```

### 如何查看客户端侧的错误？ <a href="#how-do-i-see-errors-on-the-client-side" id="how-do-i-see-errors-on-the-client-side"></a>

请看一下我们的前端示例。Passwordless.dev JavaScript 函数返回一个包含令牌或错误：`{ token: string, error: ProblemDetails}` 的对象。

```typescript
const { token, error } = p.signinWithDiscoverable();
if(error) {
  console.error(error); // Console logs a problem details object.
}
```

## 错误代码列表 <a href="#list-of-error-codes" id="list-of-error-codes"></a>

### invalid\_token <a href="#invalid-token" id="invalid-token"></a>

提交的令牌不包含预期值。这种错误通常发生在流程早期出错时，发送的是错误信息而不是有效令牌。

当调用需要某种类型（`verify_` 或 `register_`）令牌的端点时，您将收到此错误。

#### **解决方法** <a href="#solution" id="solution"></a>

确保您的请求包含令牌的预期值。

### missing\_register\_token <a href="#missing-register-token" id="missing-register-token"></a>

当您调用 `p.register(registerToken)` 但 `registerToken` 的值为空或无效时，您将收到此错误。

您传递给前端 `p.register(token)` 的令牌无效。您的后端调用生成的注册令牌可能失败，随后反馈一个错误响应 `.register()` 调用而不是令牌。

另请参阅 [invalid\_token](errors.md#invalid-token)。

#### **解决方法** <a href="#solution" id="solution"></a>

确保您的 `registerToken` 中包含预期值。通过调用 `/register/token` 端点从后端获取注册令牌。它应该以 `register_` 开头。

### unknown\_credential <a href="#unknown-credential" id="unknown-credential"></a>

当您调用 `p.signinWith*()` 但所使用的通行密钥未在我们的系统中注册时，您将收到此错误。如果通行密钥已在服务器上删除（例如，在应用程序 UI 或管理控制台中将其删除）但仍然存在于用户设备上，则可能会发生这种情况。

#### **解决方法** <a href="#solution" id="solution"></a>

从用户设备中删除通行密钥。这可以在浏览器的设置或操作系统的凭据管理器中完成。

### missing\_userid <a href="#missing-userid" id="missing-userid"></a>

当您调用 `/register/token` 端点但无法在 `json` 负载中提供 userId 时，您将收到此错误。

#### **解决方法** <a href="#solution" id="solution"></a>

创建 `register_token` 时，您必须提供有效的 userId。将 userId 添加到您的负载中，例如：

```json5
{
  "userId": "123",
  "username": "pjfry@passwordless.dev"
}
```

### expired\_token <a href="#expired-token" id="expired-token"></a>

您尝试使用的令牌已过期。默认情况下，令牌的有效期仅为 120 秒。当您尝试使用令牌执行以下操作时，您可能会收到此错误：

* 注册通行密码
* 验证登录

#### **解决方法** <a href="#solution" id="solution"></a>

确保在创建令牌后立即使用它们。令牌的有效期很短。如果使用情况需要，可以在创建令牌时配置非默认的过期时间。

### alias\_conflict <a href="#alias-conflict" id="alias-conflict"></a>

您尝试使用的别名已被其他 userId 使用。

#### **解决方法** <a href="#solution" id="solution"></a>

您需要为每个 userId 使用唯一的别名。您可以通过调用 `/alias` 端点从现有用户中删除别名。[了解更多](api.md#alias)。

### missing\_token <a href="#missing-token" id="missing-token"></a>

该端点需要一个令牌，但令牌不存在。

另请参阅 [invalid\_token](errors.md#invalid-token)。

### invalid\_token\_format <a href="#invalid-token-format" id="invalid-token-format"></a>

当使用令牌调用端点但格式未正确使用 base64url 编码时，您将收到此错误。正常情况下，您应该不会遇到这个问题，也不需要关心编码问题。

另请参阅 [invalid\_token](errors.md#invalid-token)。

### invalid\_attestation <a href="#invalid-attestation" id="invalid-attestation"></a>

如果您在调用 `/register/token` 时尝试使用除 `"none"` 之外的验证格式，您将收到这些错误。

目前，Passwordless.dev 仅支持 `"none"` 验证格式。虽然这适用于大多数应用程序，但如果您迫切要求使用 `"direct"` 或 `"indirect"` 验证，请联系我们的支持部门 [support@passwordless.dev](mailto:support@passwordless.dev) 讨论各种可能性。

#### **解决方法** <a href="#solution" id="solution"></a>

当您调用 `/register/token` 端点时，请移除 `attestation` 属性或将值更改为 `"none"`。

### max\_users\_reached <a href="#max-users-reached" id="max-users-reached"></a>

当您尝试创建新的用户但已达到计划的最大用户数时，您会收到此错误。[了解更多](https://bitwarden.com/products/passwordless/)。
