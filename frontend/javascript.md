# JavaScript 客户端

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/frontend/javascript.html)
{% endhint %}

您的前端使用 Passwordless.dev JavaScript 客户端来完成与浏览器的 FIDO2 WebAuthn 加密交换。

所有方法都**要求**使用您的 API [公钥](../concepts.md#api-keys)进行身份验证。对[私有 API](../api.md) 发出的请求则需要您的 API [私有机密](../concepts.md#api-keys)。

## 安装 <a href="#installation" id="installation"></a>

要安装 Passwordless.dev JavaScript 客户端：

{% tabs %}
{% tab title="yarm" %}
```bash
yarn add @passwordlessdev/passwordless-client
```

在所有情况下，您的前端都必须导入库才能调用本文档中的方法：

```javascript
import { Client } from '@passwordlessdev/passwordless-client';
const p = new Client({ apiKey: "" })
```
{% endtab %}

{% tab title="npm" %}
```bash
npm install @passwordlessdev/passwordless-client
```

在所有情况下，您的前端都必须导入库才能调用本文档中的方法：

```javascript
import { Client } from '@passwordlessdev/passwordless-client';
const p = new Client({ apiKey: "" })
```
{% endtab %}

{% tab title="ES6" %}
在所有情况下，您的前端都必须导入库才能调用本文档中的方法：

```html
<script type="module">
import { Client } from "https://cdn.passwordless.dev/dist/1.1.0/esm/passwordless.min.mjs"
const p = new Client({ apiKey: "" })
</script>
```
{% endtab %}

{% tab title="html" %}
```html
<script src="https://cdn.passwordless.dev/dist/1.1.0/umd/passwordless.umd.min.js" crossorigin="anonymous"></script>
```

在所有情况下，您的前端都必须导入库才能调用 Passwordless.dev 使用的方法：

```html
<script>
const Client = Passwordless.Client;
const p = new Client({});
</script>
```
{% endtab %}
{% endtabs %}

## .register() <a href="#register" id="register"></a>

调用 `.register()` 方法从后端获取一个[注册令牌](../concepts.md#tokens)，以授权在最终用户的设备上创建一个通行密钥，例如：

```javascript
// 使用您的 API 公钥实例化无密码客户端。
const p = new Passwordless.Client({
    apiKey: "myapplication:public:4364b1a49a404b38b843fe3697b803c8"
});

// 从后端获取注册令牌。
const backendUrl = "https://localhost:8002";
const registerToken = await fetch(backendUrl + "/create-token?userId" + userId).then(r => r.json());

// 在最终用户的设备上注册令牌。
const { token, error } = await p.register(registerToken);
```

成功实施后，将提示 Passwordless.dev 通过用户的网页浏览器 API 协商创建一个通行密钥，并将其公钥保存到数据库中以供将来的登录操作。

## .signinWith() <a href="#signinwith" id="signinwith"></a>

调用 `.signin` 方法生成一个[身份验证令牌](../concepts.md#tokens)，后端将检查该令牌以完成登录。有几种不同的 `.signinWith*()` 方法可用：

| 方法                          | 描述                                        | 示例                                                 |
| --------------------------- | ----------------------------------------- | -------------------------------------------------- |
| `.signinWithAutofill()`     | 触发浏览器原生自动填充 UI 以选择身份然后登录。                 | `verify_token = await p.signinWithAutofill();`     |
| `.signinWithDiscoverable()` | 触发浏览器原生提示 UI 以选择身份然后登录。                   | `verify_token = await p.signinWithDiscoverable();` |
| `.signinWithAlias(alias)`   | 使用[别名](../api.md#alias)（例如电子邮件、用户名）来指定用户。 | `verify_token = await p.signinWithAlias(email);`   |
| `.signinWithId(id)`         | 使用 userId 来指定用户。                          | `verify_token = await p.signinWithId(userId);`     |

```javascript
// 使用 API 公钥实例化一个无密码客户端。
const p = new Passwordless.Client({
    apiKey: "myapplication:public:4364b1a49a404b38b843fe3697b803c8"
});


// 为用户生成身份验证令牌。

// 选项 1：使浏览器能够为任何具有 autofill="webauthn" 的输入建议密码（仅适用于可发现的密码）。
const { token, error } = await p.signinWithAutofill();

// 选项 2：使浏览器能够通过打开 UI 提示来建议密钥（仅适用于可发现的密钥）。
const { token, error } =  await p.signinWithDiscoverable();

// 选项 3：使用用户指定的别名。
const email = "pjfry@passwordless.dev";
const { token, error } = await p.signinWithAlias(email);

// 选项 4：如果已知，则使用 userId，例如，如果用户正在重新进行身份验证。
const userId = "107fb578-9559-4540-a0e2-f82ad78852f7";
const { token, error } = await p.signinWithId(userId);

if(error) {
    console.error(error);
    // { errorCode: "unknown_credential", "title": "That credential is not registered with this website", "details": "..."}
}

// 调用您的后端来验证令牌。
const backendUrl = "https://localhost:8002"; // 您的后端
const verifiedUser = await fetch(backendUrl + "/signin?token=" + token).then(r => r.json());
if(verifiedUser.success === true) {
  // 如果成功，继续！
  // verifiedUser.userId = "107fb578-9559-4540-a0e2-f82ad78852f7";
}
```

### 响应 <a href="#response" id="response"></a>

所有 `.signinWith*()` 方法都会返回一个具有两个属性的对象，通常解构为：

```javascript
// 解构
const { token, error } = await p.siginWithId(123)

// 普通对象
const signinResponse = await p.signinWithId(123);
console.log(signinResponse.token) // "verify_xxyyzz"
console.log(signinResponse.error) // 未定义或问题详细信息对象
```

如果登录成功，此 `token` 将具有一个字符串值 `"verify_xxyyzz"`。如果登录失败，此 `error` 属性将包含[问题详细信息](../errors.md#problem-details)。

## .isBrowserSupported() <a href="#isbrowsersupported" id="isbrowsersupported"></a>

调用静态 `.isBrowserSupported()` 方法来检查最终用户的浏览器是否支持基于通行密码的 FIDO2 WebAuth 身份验证，例如：

```javascript
if (Passwordless.isBrowserSupported()) {

}
```

## .isPlatformSupported() <a href="#isplatformsupported" id="isplatformsupported"></a>

调用静态异步 `.isPlatformSupported()` 方法来检查最终用户设备或浏览器是否支持 Windows Hello 等平台身份验证，例如：

```javascript
if (await Passwordless.isPlatformSupported()) {

}
```

## .isAutofillSupported() <a href="#isautofillsupported" id="isautofillsupported"></a>

调用静态异步 `.isAutofillSupported()` 方法来检查最终用户的设备或浏览器是否支持在自动填充对话框中列出通行密钥，例如：

```javascript
if (await Passwordless.isAutofillSupported()) {

}
```
