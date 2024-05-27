# =应用程序

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/admin-console/applications.html)
{% endhint %}

您可以创建的应用程序数量取决于您的[计划](https://bitwarden.com/products/passwordless/#pricing)。选择一个应用程序以查看以下组件：

## 入门 <a href="#get-started" id="get-started"></a>

**入门**页面将引导您完成在应用程序中运行 Passwordless.dev 的初步步骤。此信息与[入门](../get-started.md)指南中记录的信息非常相似。

{% hint style="warning" %}
此页面包含您的 **API 密钥**。将 API 密钥下载到安全的地方非常重要，因为它们会在 7 天后从管理控制台中被移除。
{% endhint %}

## Playground <a href="#playground" id="playground"></a>

通过 **Playground** 页面，您可以访问一个简单的 Passwordless 演示，用于测试设备。

## 用户 <a href="#users" id="users"></a>

通过**用户**页面，您可以监控为应用程序注册了通行密钥的终端用户。对于每个用户，根据他们的 userId，您将能够查看：

### **凭据** <a href="#credentials" id="credentials"></a>

列出了每个用户已注册的凭据。[了解每个凭证存储了哪些数据](../concepts.md#credential)。

### **别名** <a href="#aliases" id="aliases"></a>

列出了每个用户已注册的别名，但此处无法查看经过哈希处理的别名（[了解更多](../api.md#alias)）。

## 设置 <a href="#settings" id="settings"></a>

**设置**页面将提供一些用于配置您的应用程序的选项，包括您的应用程序所使用的[计划](https://bitwarden.com/products/passwordless/#pricing)。未来将提供更多。

### API 密钥管理 <a href="#api-key-management" id="api-key-management"></a>

### 手动生成的身份验证令牌 <a href="#manually-generated-authentication-tokens" id="manually-generated-authentication-tokens"></a>

### 魔法链接 <a href="#magic-links" id="magic-links"></a>
