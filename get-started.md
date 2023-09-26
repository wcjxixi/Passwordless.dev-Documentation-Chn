# 入门

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/get-started.html)
{% endhint %}

Passwordless.de 是一个软件工具包，它可以帮助网络开发人员创建应用程序，使用 [FIDO2 WebAuthn 通行密钥](concepts.md)轻松验证终端用户的身份。在本指南中，我们将为您的网站指明实现 Passwordless.de 的最快途径。

{% hint style="success" %}
本指南将跳过一些概念性基础知识，以便让你尽快上手。请查看[概念](concepts.md)，以深入了解 Passwordless.dev 使用的理念。
{% endhint %}

在本指南中，我们将提供 JavaScript 示例，您也可以在[后端语言示例](backend/)和[前端框架示例](frontend/)中查看其他工具包的示例代码、指南和提示。

## 注册 <a href="#sign-up" id="sign-up"></a>

[注册](https://admin.passwordless.dev/signup)免费的 Passwordless.dev 账户。Bitwarden 提供免费的 Passwordless.de 账户，或解锁某些使用级别和功能的[付费计划](https://bitwarden.com/products/passwordless/#pricing)。

注册后，您将进入管理控制台，这是创建和配置应用程序、监控应用程序使用情况和管理计费的主要图形用户界面 (GUI)：

{% embed url="https://docs.passwordless.dev/assets/img/admin-console.0ab2087f.png" %}

## 创建应用程序 <a href="#create-an-application" id="create-an-application"></a>

选择**创建应用程序**按钮，并为新的应用程序提供**应用程序名称**和**描述**。将为每一个应用程序生成一组 [API 密钥](concepts.md#api-keys)。您将使用这些 API 密钥对 Passwordless.dev API 进行身份验证。把你的公钥和私钥保存在安全的地方，比如 [Bitwarden Secrets Manager](https://help.ppgg.in/secrets-manager/secrets-manager-overview)。

{% embed url="https://docs.passwordless.dev/assets/img/get-started_1.a7a2047b.png" %}

{% embed url="https://docs.passwordless.dev/assets/img/get-started_2.eb5ac391.png" %}

{% embed url="https://docs.passwordless.dev/assets/img/get-started_3.6e0708d4.png" %}

{% hint style="warning" %}
将 API 密钥下载到安全的地方非常重要，因为它们会在 7 天后从管理控制台中删除。
{% endhint %}

## 安装库 <a href="#install-the-library" id="install-the-library"></a>

接下来，安装 [Passwordless.dev JavaScript 客户端库](frontend/javascript.md)，可以全局安装，也可以作为应用程序中的一个模块安装。该库将允许您的应用程序与 Passwordless.dev API 和浏览器的 WebAuthn API 进行交互。要安装该库：

{% tabs %}
{% tab title="yarm" %}
```bash
yarn add @passwordlessdev/passwordless-client
```

在任何情况下，您的前端都必须导入该库，才能调用 Passwordless.dev 所使用的方法：

```javascript
import { Client } from '@passwordlessdev/passwordless-client';
```
{% endtab %}

{% tab title="npm" %}
```bash
npm install @passwordlessdev/passwordless-client
```

在任何情况下，您的前端都必须导入该库，才能调用 Passwordless.dev 所使用的方法：

```javascript
import { Client } from '@passwordlessdev/passwordless-client';
```
{% endtab %}

{% tab title="ES6" %}
```html
<script src="https://cdn.passwordless.dev/dist/1.1.0/esm/passwordless.min.mjs" type="module" crossorigin="anonymous"></script>
```

在任何情况下，您的前端都必须导入该库，才能调用 Passwordless.dev 所使用的方法：

```html
<script type="module">
    import { Client } from "https://cdn.passwordless.dev/dist/1.1.0/esm/passwordless.min.mjs"
</script>
```
{% endtab %}

{% tab title="html" %}
```html
<script src="https://cdn.passwordless.dev/dist/1.1.0/umd/passwordless.umd.min.js" crossorigin="anonymous"></script>
```

在任何情况下，您的前端都必须导入该库，才能调用 Passwordless.dev 所使用的方法：

```html
<script>
const Client = Passwordless.Client;
const p = new Client({});
</script>
```
{% endtab %}
{% endtabs %}

## 构建注册流程 <a href="#build-a-registration-flow" id="build-a-registration-flow"></a>

接下来，在后端和前前端实施一个注册[通行密钥](concepts.md#passkeys)的工作流程。从高层次来看，您将执行以下操作：

{% embed url="https://docs.passwordless.dev/assets/img/register-diagram.c34c4838.png" %}

让我们来分解一下这些步骤：

1、在后端，通过调用 passwordless.dev API 的 `/register/token` 端点生成一个[注册令牌](api.md#register-token)（[什么是令牌？](concepts.md#tokens)）。虽然您可以发送多个选项，但最少的参数是 `userId` 和 `username`，例如：

<mark style="background-color:orange;">后端</mark>

```javascript
// Node.js - 为此步骤编写的代码应在您的后端运行。

const payload = {
  "userId": "107fb578-9559-4540-a0e2-f82ad78852f7", // 必填。WebAuthn 用户句柄，应由您的应用程序生成。最多 64 字节。
  "username": "pjfry@passwordless.dev", // 必填。由用户选择的，用于用户身份验证的人类可读的用户名。
  // ...有关更多选项，请参阅后端 API 参考中的 /register/token。
};

// 使用您的 API 私有密机密将 payload POST 到 Passwordless.dev API。
const apiUrl = "https://v4.passwordless.dev";
const {token} = await fetch(apiUrl + "/register/token", {
    method: "POST",
    body: JSON.stringify(payload),
    headers: {
        "ApiSecret": "myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4",
        "Content-Type": "application/json"
    }
}).then(r => r.json());
```

成功实施后将创建一个注册令牌，并以字符串形式返回，例如：

```json
{ "token": "register_wWdDh02ItIvnCKT_02ItIvn..." }
```

{% hint style="success" %}
如果您的 API 请求失败，您将收到一个包含 `json` 格式的[问题详细信息](errors.md)的错误响应。
{% endhint %}

2、在前端，发起 WebAuthn 流程以使用已生成的注册令牌创建和存储通行密钥（[了解更多](frontend/javascript.md)），例如：

<mark style="background-color:orange;">前端</mark>

```javascript
// 为此步骤编写的代码应在您的前端运行。
import {Client} from "@passwordlessdev/passwordless-client";

// 使用您的 API 公钥实例化无密码客户端。
const p = new Client({
    apiKey: "myapplication:public:4364b1a49a404b38b843fe3697b803c8"
});

// 从后端获取返回的注册令牌。
const backendUrl = "https://localhost:7002"; // 您的后端
const registerToken = await fetch(backendUrl + "/create-user").then(r => r.json());

// 在最终用户的设备上注册令牌。
const {token, error} = await p.register(registerToken);
if(token) {
    // 成功注册！
} else {
    console.error(error);
}
```

成功实施后，Passwordless.de 将通过用户的网络浏览器 API 协商创建一个通行密钥，并将其公钥保存到数据库中，以备今后登录操作之用。

## 构建登录流程 <a href="#build-a-signin-flow" id="build-a-signin-flow"></a>

接下来，在后端和前前端实施一个使用[通行密钥](concepts.md#passkeys)登录的工作流程。从高层次来看，您将执行以下操作：

{% embed url="https://docs.passwordless.dev/assets/img/signin-diagram.62c64069.png" %}

您编写的代码必须：

1、在前端，发起登录并获取[验证令牌](concepts.md#tokens)，后台将检查该验证令牌以完成登录。您可以使用别名、userId 或可发现凭据（[了解更多](frontend/javascript.md#signinwith)）来发起登录：

<mark style="background-color:orange;">前端</mark>

```javascript
// 为此步骤编写的代码应在您的前端运行。

// 使用您的 API 公钥实例化无密码客户端。
const p = new Client({
    apiKey: "myapplication:public:4364b1a49a404b38b843fe3697b803c8"
});

// 允许用户指定一个用户名或别名。
const alias = "pjfry@passwordless.dev";

// 为用户生成验证令牌。
const {token, error} = await p.signinWithAlias(alias);
// 提示：您也可以尝试使用 p.signinWithDiscoverable();

// 调用您的后端来验证已生成的令牌。
const backendUrl = "https://localhost:7002"; // 您的后端
const verifiedUser = await fetch(backendUrl + "/signin?token=" + token).then(r => r.json());
if(verifiedUser.success === true) {
  // 如果成功，继续！
}
```

成功实施后，后端将获得验证令牌。在上面的示例中，客户端等待后端返回 `true`（**步骤 2**），然后再继续对已确认的登录采取操作。

2、通过使用已生成的令牌调用 Passwordless.dev API 的 `/signin/verify` 端点（了解更多）来验证验证令牌，例如：

<mark style="background-color:orange;">后端</mark>

```javascript
// 为此步骤编写的代码应在您的后端运行。
// 从前端获取验证令牌。
const token = { token: req.query.token };

// 使用您的 API 私有机密将验证令牌 POST 到 Passwordless.dev API。
const apiUrl = "https://v4.passwordless.dev";
const response = await fetch(apiurl + "/signin/verify", {
    method: "POST",
    body: JSON.stringify({token}),
    headers: { "ApiSecret": "myapplication:secret:11f8dd7733744f2596f2a28544b5fbc4", "Content-Type": "application/json" }
});

// 将 API 响应（见下文）缓存到变量。
const body = await response.json();

// 检查 API 响应是否成功验证。
// 要查看此端点返回的所有属性，请参阅后端 API 参考中的 /signin/verify。
if (body.success) {
    console.log("Successfully verified sign-in for user.", body);
    // 设置一个 cookie/userid。
} else {
    console.warn("Sign in failed.", body);
}
```

成功实施后上述的 `POST` 后，将返回一个成功的响应，其中包括用户的 `userId`，例如：

```json
{
  "success": true,
  "userId": "123",
  "timestamp": "2021-08-01T01:33:36.9773187Z",
  "rpid": "example.com",
  "origin": "http://example.com:3000",
  "device": "Chrome, Windows 10",
  "country": "",
  "nickname": "Home laptop",
  "credentialId": "Mq1ZhrHBmhly34YaO/uuXuNuf/VCHDkuknENz/LZJR4=",
  "expiresAt": "2021-08-01T01:35:36.9773193Z"
}
```

使用 `.success` 值（`true` 或 `false`）确定下一步操作，例如是否通过设置 cookie 等方式来完成登录（**步骤 1**）。

## 下一步 <a href="#next-steps" id="next-steps"></a>

恭喜您已经掌握了 Passwordless.dev 的基本实现！接下来：

* 查看其他[后端语言](backend/)和[前端框架](frontend/)，找到最适合您的应用程序的语言。
* 深入了解[管理控制台](admin-console.md)提供的功能。
* 找出适合您的应用程序或业务需求的[最佳方案](https://bitwarden.com/products/passwordless/#pricing)。
