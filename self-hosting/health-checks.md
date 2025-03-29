# 健康检查

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/self-hosting/health-checks.html)
{% endhint %}

我们已向 API 添加了一些健康检查端点，以便您可以监控 AdminConsole 和 API 的状态。

## API <a href="#api" id="api"></a>

### 简单 <a href="#simple" id="simple"></a>

简单的健康检查端点不会检查任何依赖项，仅验证给定的端点是否可达。您可以使用它来检查您的容器是否仍在运行，或者验证您的端口映射或 DNS 配置。

```sh
curl https://yourdomain.com:5701/api/health/http
```

如果 API 正在运行，并且正文为 `Healthy`，将返回一个 `200 OK`。

### 数据库 <a href="#database" id="database"></a>

要验证您的数据库配置是否正确完成，您可以使用数据库健康检查端点。此端点将验证数据库是否可访问，以及数据库模式是否是最新的。

```sh
curl https://yourdomain.com:5701/api/health/storage
```

将作为以下响应返回 `200 OK`。当数据库配置不正确或不可访问时，将返回 `503 Service Unavailable`。

作为成功的响应，您可以在 `results` 下找到以下键：

* `orm`：与 Microsoft 的 ORM（叫做实体框架）相关。如果失败，可能是因为数据库模式未更新。
* `sqlite`：与 Sqlite 数据库相关。如果失败，可能是因为数据库文件不可访问。
* `mssql`：与 Microsoft SQL Server 相关。如果失败，可能是因为数据库不可访问。

```json
{
  "status": "Healthy",
  "elapsedMilliseconds": 72.8363,
  "results": {
    "orm": {
      "status": "Healthy",
      "elapsedMilliseconds": 72.3092,
      "data": {}
    },
    "sqlite": {
      "status": "Healthy",
      "elapsedMilliseconds": 12.8241,
      "data": {}
    }
  }
}
```

### E-mailing <a href="#e-mailing" id="e-mailing"></a>

为了验证您的电子邮箱配置是否正确完成，您可以使用电子邮箱健康检查端点。此端点将验证电子邮箱服务器是否可达，以及电子邮箱凭据是否正确。

```sh
curl https://yourdomain.com:5701/api/health/mail
```

#### 默认（未配置） <a href="#default-not-configured" id="default-not-configured"></a>

默认情况下，如果没有提供 SMTP 或 PostMark 配置，所有电子邮箱都将写入容器内的文件中。

您可以通过查看 `results` 键来识别这一点，该键将包含以下内容：

```json
{
  "status": "Healthy",
  "elapsedMilliseconds": 1.8225,
  "results": {}
}
```

如果 `results` 键不包含任何子键，例如 `smtp` 或 `postmark`，则表示未提供电子邮箱配置。

一个有效的 `smtp` 配置将返回如下响应：

```json
{
  "status": "Healthy",
  "elapsedMilliseconds": 1.8225,
  "results": {
    "smtp": {
      "status": "Healthy",
      "elapsedMilliseconds": 1.4473,
      "data": {}
    }
  }
}
```

而一个无效的 smtp 配置将返回 `503 Service Unavailable`。
