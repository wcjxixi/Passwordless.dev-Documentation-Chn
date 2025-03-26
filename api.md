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

{% tab title="JS" %}
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

// 使用您的 API 私有机密将 payload POST 到 Passwordless.dev API。
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
| `aliases`           | userId 的别名数组，例如电子邮件或用户名。用于使用 `signinWithAlias()` 方法在客户端发起登录。别名对于 userId 必须是唯一的。默认为空数组 `[]`。                                     | `["pjfry@passwordless.dev"]`             |
| `aliasHashing`      | 别名在存储之前是否应进行哈希处理。默认为 `true`。                                                                                                    | `true`                                   |

### 响应 <a href="#response" id="response"></a>

成功后，`/register/token` 端点将创建一个以 json 格式返回的注册令牌，例如：

```json
{ "token": "register_wWdDh02ItIvnCKT_02ItIvn..."}
```

您的前端将使用此注册令牌来协商 WebAuth 凭据的创建。

## /signin/verify <a href="#signin-verify" id="signin-verify"></a>

### 请求 <a href="#request" id="request"></a>

向 `/signin/verify` 端点发出的 `POST` 请求会解压[身份验证令牌](concepts.md#tokens)，该令牌必须通过在前端调用 `.signinWith*()` 方法来生成（[了解更多](frontend/javascript.md#signinwith)）并包含在请求正文中，例如：

{% tabs %}
{% tab title="HTTP" %}
```http
POST https://v4.passwordless.dev/signin/verify HTTP/1.1
ApiSecret: myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4
Content-Type: application/json

{
  "token": "d5vzCkL_GvpS4VYtoT3..."
}
```
{% endtab %}

{% tab title="JS" %}
```javascript
const apiUrl = "https://v4.passwordless.dev";

// 从前端获取身份验证令牌。
const payload = { token: req.query.token };

// 使用您的 API 私有机密将身份验证令牌 POST 到 Passwordless.dev API。
const response = await fetch(apiUrl + "/signin/verify", {
    method: "POST",
    body: JSON.stringify(payload),
    headers: { "ApiSecret": "myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4", "Content-Type": "application/json" }
});
```
{% endtab %}
{% endtabs %}

Passwordless.dev 私有 API 将解压身份验证令牌以检查其合法性。

### 响应 <a href="#response" id="response"></a>

成功后，`/signin/verify` 端点将返回一个成功的响应对象，例如：

```json
{
  "success": true,
  "userId": "123",
  "timestamp": "3023-08-01T14:43:03Z",
  "rpid": "localhost",
  "origin": "http://localhost:3000",
  "device": "Firefox, Windows 10",
  "country": "SE",
  "nickname": "My Work Phone",
  "expiresAt": "3023-08-01T14:43:03Z",
  "tokenId": "TODO",
  "type": "passkey_signin" // or passkey_register  
}
```

使用 `.success` 值（`true` 或 `false`）确定下一步操作，即是否完成登录（[了解更多](frontend/javascript.md#signinwith)）。

## /signin/generate-token <a href="#signin-generate-token" id="signin-generate-token"></a>

### 请求 <a href="#request" id="request"></a>

向 `/signin/generate-token` 端点发出的 `POST` 请求可为用户手动生成一个身份验证令牌，从而避开常规的登录流程（即 `.signinWith*()` 方法）。生成的令牌可以通过 `/signin/verify` 端点进行验证，并像普通身份验证令牌一样使用。

{% tabs %}
{% tab title="HTTP" %}
```http
POST https://v4.passwordless.dev/signin/generate-token HTTP/1.1
ApiSecret: myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4
Content-Type: application/json

{
  "userId": "123"
}
```
{% endtab %}

{% tab title="JS" %}
```javascript
const apiUrl = 'https://v4.passwordless.dev';

// 生成身份验证令牌，绕过通常的登录过程。
const payload = {
  userId: '107fb578-9559-4540-a0e2-f82ad78852f7'
};

// 使用您的 API 私有机密将用户 ID POST 到 Passwordless.dev API。
const response = await fetch(apiUrl + '/signin/generate-token', {
  method: 'POST',
  body: JSON.stringify(payload),
  headers: {
    'ApiSecret': 'myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4',
    'Content-Type': 'application/json'
  }
});
```
{% endtab %}
{% endtabs %}

### 相应 <a href="#response" id="response"></a>

成功后，`/signin/generate-token` 端点将返回一个响应对象，例如：

```json
{
  "token": "d5vzCkL_GvpS4VYtoT3..."
}
```

## /alias <a href="#alias" id="alias"></a>

### 请求 <a href="#request" id="request"></a>

向 `/alias` 端点发出的 `POST` 请求会根据他们的 `userId` 向用户添加别名（[了解更多](concepts.md#user-verification)），以便允许使用其他用户名、电子邮件地址等登录。

请求正文**必须包含**用户的 `userId` 和**完整的**别名数组，因为发出 `POST` 请求时预先存在的别名将被覆盖，例如：

{% tabs %}
{% tab title="HTTP" %}
```http
POST https://v4.passwordless.dev/alias HTTP/1.1
ApiSecret: myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4
Content-Type: application/json

{
  "userId": "107fb578-9559-4540-a0e2-f82ad78852f7",
  "aliases": [
    "pjfry@passwordless.dev",
    "benderrules@passwordless.dev"
  ],
  "hashing": true
}
```
{% endtab %}

{% tab title="JS" %}
```javascript
const apiUrl = "https://v4.passwordless.dev";

// 获取用户现有的和新的别名数组。
const payload = {
    "userId": "107fb578-9559-4540-a0e2-f82ad78852f7",
    "aliases": ["pjfry@passwordless.dev", "benderrules@passwordless.dev"],
    "hashing": true // 存储前是否对别名进行哈希处理，默认为 true。
};

// 使用您的 API 私有机密将数组 POST 到 Passwordless.dev API。
await fetch(apiUrl + "/alias", {
    "method": "POST",
    "body": JSON.stringify(payload),
    "headers": { "ApiSecret": "myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4", "Content-Type": "application/json"}
});
```
{% endtab %}
{% endtabs %}

{% hint style="success" %}
默认情况下，`"hashing": true` 将在存储别名之前打开，以对其进行哈希处理。您将无法在管理控制台中查看已哈希的别名。
{% endhint %}

允许用户创建别名时需要考虑的一些规则：

* 别名对于指定的 `userId` 必须是唯一的。
* 别名长度不得超过 250 个字符。
* 一个 `userId` 最多可以有 10 个与其关联的别名。

### 响应 <a href="#response" id="response"></a>

任何 API 响应中都不会返回别名，并且可以对其进行哈希处理以保护用户隐私（见上文）。成功后，`/alias` 端点将返回 HTTP 200 OK [状态代码](api.md#status-codes)。

## /credentials/list <a href="#credentials-list" id="credentials-list"></a>

### 请求 <a href="#request" id="request"></a>

向 /`credentials/list` 端点发出的 `GET` 请求会列出与用户关联的所有[已注册凭据](concepts.md#credential)（由 `userId` 指定）。请求**必须包含**相关的 `userId`，例如：

{% tabs %}
{% tab title="HTTP" %}
```http
GET https://v4.passwordless.dev/credentials/list?userId=107fb578-9559-4540-a0e2-f82ad78852f7 HTTP/1.1
ApiSecret: myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4
```
{% endtab %}

{% tab title="JS" %}
```javascript
const apiUrl = "https://v4.passwordless.dev";

// 获取相关用户的 userId。
const payload = {
    "userId": "107fb578-9559-4540-a0e2-f82ad78852f7"
};

// 使用您的 API 私有机密将 userId POST 到 Passwordless.dev API。
const credentials = await fetch(apiUrl + "/credentials/list", {
    "method": "POST",
    "body": JSON.stringify(payload),
    "headers": { "ApiSecret": "myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4", "Content-Type": "application/json"}
}).then(r => r.json());
```
{% endtab %}
{% endtabs %}

### 响应 <a href="#response" id="response"></a>

成功后， `/credentials/list` 端点将返回 `.json` 对象数组，其中每个对象代表一个[已注册的凭据](concepts.md#credential)：

```json
[
  {
    "descriptor": {
      "type": "public-key",
      "id": "2mgrJ6LPItfxbnVc2UgFPHowNGKaYBm3Pf4so1bsXSk"
    },
    "publicKey": "pQECAyYgASFYIPi4M0A+ZFeyOHEC9iMe6dVhFnmOZdgac3MRmfqVpZ0AIlggWZ+l6+5rOGckXAsJ8i+mvPm4YuRQYDTHiJhIauagX4Q=",
    "userHandle": "YzhhMzJlNWItNDZkMy00ODA4LWFlMTAtMTZkM2UyNmZmNmY5",
    "signatureCounter": 0,
    "createdAt": "2023-04-21T13:33:50.0764103",
    "aaGuid": "adce0002-35bc-c60a-648b-0b25f1f05503",
    "lastUsedAt": "2023-04-21T13:33:50.0764103",
    "rpid": "myapp.example.com",
    "origin": "https://myapp.example.com",
    "country": "US",
    "device": "Chrome, Mac OS X 10",
    "nickname": "Fred's Macbook Pro 2",
    "userId": "c8a32e5b-46d3-4808-ae10-16d3e26ff6f9"
  } //, ...
]
```

[详细了解这些键值对的含义](concepts.md#credential)。

## /credentials/delete <a href="#credentials-delete" id="credentials-delete"></a>

### 请求 <a href="#request" id="request"></a>

向 `/credentials/delete` 端点发出的 `POST` 请求会删除与用户关联的特定凭证（由 `credentialId` 指定）。该请求**必须包含**相关的 `credentialId`，例如：

```http
POST https://v4.passwordless.dev/credentials/delete HTTP/1.1
ApiSecret: myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4
Content-Type: application/json

{
  "credentialId": "qgB2ZetBhi0rIcaQK8_HrLQzXXfwKia46_PNjUC2L_w"
}
```

### 响应 <a href="#response" id="response"></a>

成功后，`/credentials/delete` 端点将返回 HTTP 200 OK [状态代码](api.md#status-codes)。

## /magic-links/send <a href="#magic-links-send" id="magic-links-send"></a>

### 请求 <a href="#request" id="request"></a>

向 `/magic-links/send` 端点发出的 `POST` 请求会通过 Magic Link 向提供的地址发送电子邮件。此 Magic Link 包含您提供的 URL，该 URL 会将收件人重定向到您的应用程序中的端点。从这里，您可以发送 Passwordless.dev 嵌入链接中的令牌来验证 `signin/verify` 处的令牌。

{% hint style="warning" %}
Passwordless.dev 不存储用户电子邮件。集成 Magic Links 时，您应该验证电子邮件和用户 ID 是否属于同一用户。否则，您可能会在应用程序中引入安全漏洞。
{% endhint %}

该请求必须包含所有三个字段。

* `emailAddress`：Magic Link 的接收者。必须是一个有效的 E-mail 地址。
* `urlTemplate`：这是用户单击链接时将被定向到的 URL。它应该是除令牌模板字符串 `$TOKEN` 之外的有效 URL。在发送电子邮件之前，我们会将 `$TOKEN` 与实际令牌值交换。在您的应用程序中，您应该从 URL 中解析令牌（最容易通过查询参数完成，如下所示）并将其发送到 `signin/verify` 端点以验证请求。
* `userId`：电子邮件目标用户的标识符。
* `timeToLive`：（可选）魔术链接令牌有效的秒数。如果未设置，则默认值为 1 小时。

```http
POST https://v4.passwwordless.dev/magic-links/send HTTP/1.1
ApiSecret: myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4
Content-Type: application/json

{
  "emailAddress": "user-email@example.com",
  "urlTemplate": "https://www.myapp.com?token=$TOKEN"
  "userId": "c8a32e5b-46d3-4808-ae10-16d3e26ff6f9"
  "timeToLive": 3600
}
```

### 响应 <a href="#response" id="response"></a>

如果成功， `/magic-links/send` 端点将返回 HTTP 204（无内容）[状态代码](api.md#status-codes)。

如果尚未启用 Magic Link， `/magic-links/send` 端点将返回 HTTP 403（未经授权）[状态代码](api.md#status-codes)以及有关启用 Magic Link 功能的消息。

## /auth-configs/list <a href="#auth-configs-list" id="auth-configs-list"></a>

### 请求 <a href="#request" id="request"></a>

### 响应 <a href="#response" id="response"></a>

/auth-configs/add


### 请求 <a href="#request" id="request"></a>

### 响应 <a href="#response" id="response"></a>

/auth-configs/


### 请求 <a href="#request" id="request"></a>

### 响应 <a href="#response" id="response"></a>

/auth-configs/delete


### 请求 <a href="#request" id="request"></a>

### 响应 <a href="#response" id="response"></a>

## 状态代码 <a href="#status-codes" id="status-codes"></a>

API 会为每一个请求返回 HTTP 状态代码。

如果您收到错误，您还将收到[问题详细信息](errors.md#problem-details)形式的 JSON 序列化的错误摘要。有关详细信息，请参阅[错误页面](errors.md)。

| HTTP 代码 | 消息                                                  | 状态 |
| ------- | --------------------------------------------------- | -- |
| 200     | 一切正常。                                               | ✅  |
| 201     | 一切正常，资源已创建。                                         | ✅  |
| 204     | 一切正常，响应为空。                                          | ✅  |
| 400     | 错误请求（详情请参阅[问题详细信息](errors.md#problem-details)）。     | 🔴 |
| 401     | 您没有表明自己的身份。                                         | 🔴 |
| 403     | 您无权执行该操作（详情请参阅[问题详细信息](errors.md#problem-details)）。 | 🔴 |
| 409     | 冲突（详情请参阅[问题详细信息](errors.md#problem-details)）。       | 🔴 |
| 429     | 请求次数过多（详情请参阅[问题详细信息](errors.md#problem-details)）。   | 🔴 |
| 500     | 我们这边出了很大的问题。                                        | 🔴 |
