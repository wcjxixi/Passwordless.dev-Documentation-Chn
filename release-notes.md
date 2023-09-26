# 发行说明

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/releasenotes.html)
{% endhint %}

## 正式发布 <a href="#general-availability" id="general-availability"></a>

Bitwarden 很荣幸地宣布 Passwordless.dev 正式发布！GA 的改进包括：

* 全新的[管理控制台](admin-console.md)，帮助您快速开始实施无密码、配置设置和监控用户。
* 对 [Javascript 客户端](frontend/javascript.md)和 [API](api.md) 的一系列更新。

{% hint style="warning" %}
**对于 Beta 版用户而言**，passwordless.dev 的 GA 版本包括对 API 的重大更改，这些更改将发布到全新服务中。所有 Beta 用户都会尽快[注册 GA 服务](release-notes.md)，并对实施进行以下更改：

* 将现有的 API 密钥替换为新创建的密钥。
* 将 Beta 服务 API URL 替换为 GA 服务 URL。
* 根据文档更新您的[前端代码](frontend/javascript.md)和[后端代码](api.md)。

目前尚不支持从 Beta 版到 GA 服务的数据迁移，但 Beta 服务将在 GA 发布后继续提供，以便您有时间进行迁移。
{% endhint %}
