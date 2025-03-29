# 自托管

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/self-hosting.html)
{% endhint %}

{% hint style="success" %}
自托管选项是面向企业的功能，目前处于开发阶段的测试版。它旨在进行实验和探索。所有功能和配置可能在将来发生变化。请勿在生产环境中使用。
{% endhint %}

Docker 镜像将允许您在 5 分钟内完成自托管实例的设置。

## 开始使用

为了保留自托管 Passwordless 的设置和数据，建议您设置一个目录来存储生成的配置文件和数据库。否则，容器将删除数据库和配置文件。

### 安装和运行 <a href="#installation-and-running" id="installation-and-running"></a>

要开始运行，请执行以下行：

```sh
docker pull bitwarden/passwordless-self-host:stable
docker run \
  --publish 5701:5701 \
  --volume {your-host-directory}:/etc/bitwarden_passwordless \
  bitwarden/passwordless-self-host:stable
```

现在您应该能够访问自己的 `Passwordless.dev` 实例了，地址如下：

* 管理控制台：`https://localhost:5701`
* API：`https://localhost:5701/api`

从这里，您可以继续到[开始使用](../get-started.md)以设置您的组织和应用程序。

### 日志记录 <a href="#logging" id="logging"></a>

如果您想查看管理控制台和 API 的日志，可以使用以下命令：

```sh
docker exec -it {name-of-container} tail /var/log/bitwarden_passwordless/api.log
```

```sh
docker exec -it {name-of-container} tail /var/log/bitwarden_passwordless/admin.log
```

### 通信 <a href="#communication" id="communication"></a>

默认情况下，系统发送的电子邮件将使用一个文件。邮件正文将被追加到该文件中而不是发送出去。这对于创建组织和邀请管理员到管理控制台非常重要。要配置自己的电子邮件提供程序，请参阅[电子邮件配置文档](configuration.md#e-mail) 。

默认的 `mail.md` 文件位置如下：

* `/app/Admin/mail.md`
* `/app/Api/mail.md`

要访问这些文件，请执行进入运行中的容器或使用以下两个命令来打印文件内容。

对于管理控制台：

```sh
docker exec -it {name-of-container} cat /app/AdminConsole/mail.md
```

对于 API：

```sh
docker exec -it {name-of-container} cat /app/Api/mail.md
```

## 可用标签 <a href="#available-tags" id="available-tags"></a>

* `stable` - 从最新稳定版本发布构建（推荐）
* `1.0.55` - 从特定稳定版本发布构建
* `latest` - 从 `main` 分支的当前状态构建

请参阅 [Docker Hub](https://hub.docker.com/r/bitwarden/passwordless-self-host/tags) 页面获取更多信息。

## 已知问题 <a href="#known-issues" id="known-issues"></a>

* 自托管管理控制台与云托管管理控制台之间没有活跃的同步。.
* 您无法将自托管的组织升级到新的更高等级计划。

## 更多信息 <a href="#more-information" id="more-information"></a>

如需进一步配置选项，请参阅以下页面。

* [配置](configuration.md)
* [本地运行](running-locally.md) <mark style="background-color:yellow;">示例</mark>
* [健康检查](health-checks.md)
* [高级](advanced.md)
