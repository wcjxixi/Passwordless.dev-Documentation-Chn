# 关于 Passwordless.dev

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/)
{% endhint %}

Bitwarden Passwordless.dev 是一个软件工具包，可帮助开发人员将[基于 FIDO2 WebAuthn 的通行密钥](concepts.md#fido2)功能构建到网站和企业应用程序中，以实现无缝身份验证流程。

使用 Passwordless.dev 意味着无需阅读大量的 W3C 规范文档，无需确定要采用何种加密技术，也无需担心如何管理存储的公钥。背后的 Bitwarden 团队将为您处理这些问题。

Passwordless.dev 通过简单易用的工具为您的网站提供通行密钥验证，旨在让网站开发人员更快地采用，并应对不断变化的网络安全环境带来的挑战。

{% embed url="https://docs.passwordless.dev/assets/img/diagram.13594d21.png" %}

Passwordless.dev 的架构由三个关键部分组成：

* 一个[客户端库](frontend/javascript.md)，前端使用它来向终端用户浏览器的 WebAuthn API 发起请求，以及向 Passwordless.dev API 发起请求。
* 一个公共 RESTful API，客户端库使用它来完成与浏览器的 FIDO2 WebAuthn 密码交换。
* 一个[私有 RESTful API](api.md)，后端使用它来启动密钥注册、验证登录和为最终用户检索密钥。

Passwordless.dev 的所有代码都是[开源](https://github.com/passwordless)的。

{% hint style="success" %}
有关支持 Passwordless.dev 的原则的更多信息，请查看[概念](concepts.md)。如果您准备好开始使用 Passwordless.dev，请访问[入门](get-started.md)以获取分步说明。
{% endhint %}
