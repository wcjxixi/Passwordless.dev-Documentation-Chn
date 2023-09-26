# 后端 API 参考

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/api.html)
{% endhint %}

您的后端使用 **Passwordless.dev 私有 API** 来发起密钥注册、验证登录、为最终用户获取密钥等。

对此 API 发出的所有请求都**要求**标头中包含您的 API [私有机密](concepts.md#api-keys)以进行身份​​验证。通过 [JavaScript 客户端](frontend/javascript.md)中的方法向公共 API 发出的请求将需要您的 API [公钥](concepts.md#api-keys)。

## /register/token <a href="#register-token" id="register-token"></a>

### 请求 <a href="#request" id="request"></a>

向 `/register/token` 端点发出的 `POST` 请求会为用户创建一个[注册令牌](concepts.md#tokens)，您的前端将使用该令牌来协商 WebAuth 凭据的创建。

请求正文至少**必须包含** `userId` 和 `username`，例如：

{% tabs %}
{% tab title="HTTP" %}
```http
POST https://v4.passwordless.dev/register/token HTTP/1.1
ApiSecret: myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4
Content-Type: application/json

{
  "userId": "107fb578-9559-4540-a0e2-f82ad78852f7",
  "username": "pjfry@passwordless.dev",
  "displayname": "Philip J Fry",
}
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const apiUrl = "https://v4.passwordless.dev";

const payload = {
  "userId": "107fb578-9559-4540-a0e2-f82ad78852f7", // WebAuthn 用户句柄，应由您的应用程序生成。最多 64 字节。
  "username": "pjfry@passwordless.dev", // 用于显示目的，应帮助用户识别用户账户。
  "displayname": "Philip J. Fry", // 账户的人性化名称。
  "authenticatorType": "any", // WebAuthn 验证器附件模式。可以是 "any"（默认）、"platform"（触发特定于客户端设备的 Windows Hello、FaceID 或 TouchID 选项）或 "cross-platform"（触发如安全密钥等漫游选项）。
  "userVerification": "preferred", // 依赖方是否要求对操作进行本地调用授权。可以是 "preferred"（默认）、"required" 或 "optional"。
  "aliases": ["pjfry@passwordless.dev"], // 用户创建的标识符（如电子邮件）数组，用于引用 userId。
  "aliasHashing": true // 别名在存储之前是否应进行哈希处理。默认为 true。
};

// 使用您的 API 私有密机密将 payload POST 到 Passwordless.dev API。
const { token } = await fetch(apiUrl + "/register", {
    method: "POST",
    body: JSON.stringify(payload),
    headers: { "ApiSecret": "myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4", "Content-Type": "application/json"}
}).then(r => r.json());
```
{% endtab %}
{% endtabs %}

除了必需的参数之外，请求正文还可能包含其他参数，所有这些参数如下：

| 参数                  | 描述                                                                                                                              | 示例值                                      |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| `userId`            | **必填**。WebAuthn 用户句柄，应由您的应用程序生成。这用于识别您的用户（可以是数据库主键 ID 或 guid）。最多 64 字节。不应包含有关用户的 PII。                                           | `"107fb578-9559-4540-a0e2-f82ad78852f7"` |
| `username`          | **必填**。用户账户标的人性化识符。它仅用于显示，帮助用户确定具有相似显示名称的用户账户之间的区别。在浏览器 UI 中使用，从不存储在服务器上。                                                       | `"pjfry@passwordless.dev"`               |
| `displayname`       | 账户的人性化名称，应由用户选择。在浏览器 UI 中使用，从不存储在服务器上。                                                                                          | `"Philip J Fry"`                         |
| `attestation`       | WebAuthn 证书传递首选项。仅支持 `"none"`（默认）。                                                                                              | `"none"`（默认）                             |
| `authenticatorType` | WebAuthn 验证器附件模式。可以是 `"any"`（默认）、`"platform"`（触发特定于客户端设备的 Windows Hello、FaceID 或 TouchID 选项）或 `"cross-platform"`（触发如安全密钥等漫游选项）。 | `"any"`（默认）                              |
| `discoverable`      | 如果为 `true`，则创建客户端可发现凭据，无需用户名即可登录。                                                                                               | `true`（默认）                               |
| `userVerification`  | 允许在身份验证时选择用户验证的首选项（生物识别、PIN 码等），可以是 `"preferred"`（默认）、`"required"` 或 `"discouraged"`。                                           | `"preferred"`                            |
| `expiresAt`         | 注册令牌到期的时间戳 (UTC)。默认为当前时间 +120 秒。                                                                                                | `"3023-08-01T14:43:03Z"`                 |
| `aliases`           | userId 的别名数组，例如电子邮件或用户名。用于使用 `signinWithAlias()` 方法在客户端发起登录。别名必须与 userId 对应。默认为空数组 `[]`。                                        | `["pjfry@passwordless.dev"]`             |
| `aliasHashing`      | 别名在存储之前是否应进行哈希处理。默认为 `true`。                                                                                                    | `true`                                   |

### 响应 <a href="#response" id="response"></a>

成功后，`/register/token` 端点将创建一个以 json 格式返回的注册令牌，例如：

```json
{ "token": "register_wWdDh02ItIvnCKT_02ItIvn..."}
```

您的前端将使用此注册令牌来协商 WebAuth 凭据的创建。

## /signin/verify <a href="#signin-verify" id="signin-verify"></a>

### 请求 <a href="#request" id="request"></a>

### 响应 <a href="#response" id="response"></a>

## /alias <a href="#alias" id="alias"></a>

### 请求 <a href="#request" id="request"></a>

### 响应 <a href="#response" id="response"></a>

## /credentials/list <a href="#credentials-list" id="credentials-list"></a>

### 请求 <a href="#request" id="request"></a>

### 响应 <a href="#response" id="response"></a>

## /credentials/delete <a href="#credentials-delete" id="credentials-delete"></a>

### 请求 <a href="#request" id="request"></a>

### 响应 <a href="#response" id="response"></a>

### 状态代码 <a href="#status-codes" id="status-codes"></a>
