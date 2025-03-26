# 创建 SDK

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/backend/create-a-sdk.html)
{% endhint %}

本文提供了为 Passwordless.dev 创建官方或社区 SDK 的指南。

## 一般建议 <a href="#general-recommendations" id="general-recommendations"></a>

1. 选择开源许可证，例如 [Apache 2.0](https://choosealicense.com/licenses/apache-2.0/)
2. 在 GitHub 或 GitLab 上托管您的代码
3. 将其命名为 `passwordless-{language/framework}` （例如 `passwordless-dotnet` ）
4. 添加文档和示例（见下文）
5. 遵循目标生态系统中 SDK 的最佳实践

## 文档 <a href="#documentation" id="documentation"></a>

在您的 `README` 中，我们建议为用户提供入门所需的信息。

### 介绍 <a href="#introduction" id="introduction"></a>

* 提及这是一个社区项目
* 将用户引导至官方 Passwordless 文档：https://docs.passwordless.dev
* ...以及通过管理门户创建一个账户：https://admin.passwordless.dev

### 安装 <a href="#installation" id="installation"></a>

描述如何安装/设置 SDK。

### 入门 <a href="#getting-started" id="getting-started"></a>

提供一个基本的使用示例。

### API 参考 <a href="#api-reference" id="api-reference"></a>

记录包装 Passwordless API 的 SDK 方法。

### 示例 <a href="#examples" id="examples"></a>

包括一个创建和验证通行密钥的简单应用程序，例如如下所示的一些示例：

* [passwordless-dotnet-example](https://github.com/bitwarden/passwordless-dotnet/tree/main/examples)
* [passwordless-nodejs-example](https://github.com/bitwarden/passwordless-nodejs/tree/main/examples)

## SDK 的范围 <a href="#scope-of-the-sdk" id="scope-of-the-sdk"></a>

使用 [API 参考文档](../api.md)作为要添加的方法的蓝图。请参阅官方 SDK（例如 [passwordless-dotnet](https://github.com/bitwarden/passwordless-dotnet) 和 [passwordless-nodejs](https://github.com/bitwarden/passwordless-nodejs)）以获取方法参考以及如何实现它们。

## 测试和 CI/CD <a href="#testing-and-ci-cd" id="testing-and-ci-cd"></a>

配置自动化测试和 CI / CD（例如 GitHub Actions）以确保质量。

## 语言 SDK vs. 框架 SDK <a href="#language-sdk-vs-framework-sdk" id="language-sdk-vs-framework-sdk"></a>

我们鼓励使用不同风格的 SDK 和集成，如下所述：

* 语言 SDK：用目标语言（例如 `dotnet`、`python`、或  `ruby` 等）包装 HTTP API
* 框架 SDK：将语言 SDK 集成到固定框架（例如 ASP.NET、Flask、Rails、Nuxt 或 Laravel 等）中
* 插件：将 Passwordless.dev 作为插件集成到现有系统（例如 Wordpress、Umbraco 等）中

## 支持 <a href="#support" id="support"></a>

希望这有助于提供一些创建高质量无密码 SDK 的最佳实践！如果您有任何其他问题，请联系 team@passwordless.dev。
