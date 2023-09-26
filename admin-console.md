# 管理控制台

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/admin-console.html)
{% endhint %}

[管理控制台](https://admin.passwordless.dev/)是用于创建和配置应用程序、监控应用程序使用情况以及管理计费的主要 GUI：

{% embed url="https://docs.passwordless.dev/assets/img/admin-console.0ab2087f.png" %}

创建新应用程序后，将有几个页面可供您使用。

## 应用程序 <a href="#applications" id="applications"></a>

您可以创建的应用程序数量取决于您的[计划](https://bitwarden.com/products/passwordless/#pricing)。选择一个应用程序以查看以下组件：

### 入门 <a href="#get-started" id="get-started"></a>

**入门**页面将引导您完成在应用程序中运行 Passwordless.dev 的初步步骤。此信息与[入门](get-started.md)指南中记录的信息非常相似。

{% hint style="warning" %}
此页面包含您的 **API 密钥**。将 API 密钥下载到安全的地方非常重要，因为它们会在 7 天后从管理控制台中被移除。
{% endhint %}

### 用户 <a href="#users" id="users"></a>

通过**用户**页面，您可以监控为应用程序注册了通行密钥的终端用户。对于每个用户，根据他们的 userId，您将能够查看：

#### **凭据** <a href="#credentials" id="credentials"></a>

列出了每个用户已注册的凭据。[了解每个凭证存储了哪些数据](concepts.md#credential)。

#### **别名** <a href="#aliases" id="aliases"></a>

列出了每个用户已注册的别名，但此处无法查看经过哈希处理的别名（[了解更多](api.md#alias)）。

### 设置 <a href="#settings" id="settings"></a>

**设置**页面将提供一些用于配置您的应用程序的选项，包括您的应用程序所使用的[计划](https://bitwarden.com/products/passwordless/#pricing)。未来将提供更多。

### Playground <a href="#playground" id="playground"></a>

通过 **Playground** 页面，您可以访问一个简单的 Passwordless 演示，用于测试设备。

## 计费 <a href="#billing" id="billing"></a>

**计费**页面允许您升级为[付费组织](https://bitwarden.com/products/passwordless/#pricing)，并查看附加到您账户的应用程序列表。

## 管理员 <a href="#admins" id="admins"></a>

**管理员**页面允许您邀请其他管理员加入您的 Passwordless.dev 组织，管理应用程序、计费等。所有管理员，包括曾创建过 Passwordless.dev 账户和任何应用程序的人，**拥有组织内平相同的权限**。

{% embed url="https://docs.passwordless.dev/assets/img/admin-page.43025c60.png" %}

要邀请一位管理员：

* 在**受邀者**文本输入框中输入电子邮件地址，然后选择**发送邀请**。
* 您的未来管理员将收到一封邀请电子邮件。指示他们使用此邀请完成Passwordless.dev 的注册，然后遵循电子邮件中的验证步骤。

可以从同一页面删除管理员。

{% hint style="warning" %}
由于组织内的所有管理员都具有相同的权限，因此新邀请的管理员当前可以从组织中删除之前配置的管理员。
{% endhint %}
