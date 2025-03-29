# 配置

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/self-hosting/configuration.html)
{% endhint %}

## 端口 <a href="#ports" id="ports"></a>

容器将暴露端口 `5701`。您可以将它映射到主机上的任何端口。

`AdminConsole` 和 `Api` 都将使用端口 5000 和 5001 进行内部通信。默认情况下，网络流量不会离开容器。

## 卷 <a href="#volumes" id="volumes"></a>

您可能需要为正在运行的 Docker 容器挂载持久化存储。确保映射容器的路径 `/etc/bitwarden_passwordless` 非常重要。这样，您就可以确信您的设置和 Sqlite 数据库（假设您没有使用其他数据库），在更新、迁移等过程中保持持久化。

{% hint style="warning" %}
挂载持久化存储失败，将会：

* 如果您依赖于 `/etc/bitwarden_passwordless/config.json` ，将会导致配置丢失。
* 仅当使用 Sqlite 时，才会导致数据丢失。
{% endhint %}

```sh
$ docker run -d \
  --name passwordless \
  --mount source=/your/persistent/storage,target=/etc/bitwarden_passwordless \
  bitwarden/passwordless-self-host:stable
```

## 数据库 <a href="#database" id="database"></a>

### Sqlite <a href="#sqlite" id="sqlite"></a>

默认情况下，如果未指定其他数据库，容器将使用 Sqlite。数据将存储在以下位置：

* /etc/bitwarden\_passwordless/api.db
* /etc/bitwarden\_passwordless/admin.db

### Microsoft SQL Server <a href="#microsoft-sql-server" id="microsoft-sql-server"></a>

| 键                        | 默认值   | 必需 | 描述                                                    |
| ------------------------ | ----- | -- | ----------------------------------------------------- |
| BWP\_DB\_PROVIDER        |       | Y  | \[sqlserver/mssql] 两个值都允许您使用 Microsoft SQL Server。    |
| BWP\_DB\_SERVER          |       | Y  | 主机名，例如 'localhost' 或 'db.example.com'。                |
| BWP\_DB\_PORT            | 1433  | N  | \[0-65536]                                            |
| BWP\_DB\_DATABASE\_API   | Api   | N  | 'Api' 应用程序在 Microsoft SQL Server 实例上的数据库名称。           |
| BWP\_DB\_DATABASE\_ADMIN | Admin | N  | 'Admin Console' 应用程序在 Microsoft SQL Server 实例上的数据库名称。 |
| BWP\_DB\_USERNAME        | sa    | N  |                                                       |
| BWP\_DB\_PASSWORD        |       | Y  |                                                       |

## 环境变量 <a href="#environment-variables" id="environment-variables"></a>

| 键                        | 默认值       | 必需 | 描述                                                           |
| ------------------------ | --------- | -- | ------------------------------------------------------------ |
| BWP\_ENABLE\_SSL         | false     | N  | \[true/false] 查看以下的警告。                                       |
| BWP\_PORT                | 5701      | N  | \[0-65536] 如果您不使用反向代理，则必须设置。                                 |
| BWP\_DOMAIN              | localhost | N  | \[example.com] 此将是您自托管实例可访问的域名。它必须匹配才能正确工作这个                 |
| BWP\_DB\_PROVIDER        |           | N  | \[mssql/sqlserver/] 如果未设置默认使用 Sqlite。                        |
| BWP\_DB\_SERVER          |           | N  | 对于任何非文件托管数据库，请输入其域名。对于 Microsoft SQL Server 是必需的。            |
| BWP\_DB\_PORT            |           | N  | \[0-65536] 对于任何非文件托管数据库，请输入端口号。对于 Microsoft SQL Server 是必需的。 |
| BWP\_DB\_DATABASE\_API   |           | N  | API 的数据库名称。对于 Microsoft SQL Server 是必需的。                     |
| BWP\_DB\_DATABASE\_ADMIN |           | N  | 管理控制台的数据库名称。对于 Microsoft SQL Server 是必需的。                    |
| BWP\_DB\_USERNAME        |           | N  | 连接到数据库的用户名。对于 Microsoft SQL Server 是必需的。                     |
| BWP\_DB\_PASSWORD        |           | N  | 数据库连接用户的密码。对于 Microsoft SQL Server 是必需的。中是必需的。               |

{% hint style="warning" %}
在[不安全的环境](https://w3c.github.io/webappsec-secure-contexts/#secure-contexts)中，使用 `BWP_ENABLE_SSL` 设置 SSL 是必需的。在 'localhost' 上本地运行容器被视为是安全环境。

如果您正在使用负载均衡器或反向代理，您可以将其设置为 false，并在那里处理 SSL 终止。

在此处阅读 'WebAuthn' 规范：[查看规范](https://www.w3.org/TR/webauthn-2/#web-authentication-api)。
{% endhint %}

## E-mail <a href="#e-mail" id="e-mail"></a>

电子邮件由无密码管理控制台用于通知管理员其组织中的更改。这特别适用于在首次注册时验证管理员。

默认情况下，所有电子邮件通信都写入以下文件：

* `/app/mail.md`

使用默认配置时，以下命令将输出文件内容。

```sh
docker exec -it {name-of-container} cat /app/mail.md
```

### JSON

参考：[ASP.NET Core 中的配置](https://learn.microsoft.com/zh-cn/aspnet/core/fundamentals/configuration/)

重要的是按照以下方式配置 'Mail\_\_From'。这是为了有一个备用的电子邮件地址来发送电子邮件。在 'Mail\_\_Providers' 中有一个数组，它是一个按顺序排列的电子邮件提供程序列表，如果它们失败，我们将按顺序尝试执行。要配置电子邮件提供程序，请参阅下面的子部分。

```json
"Mail": {
  "From": "johndoe@example.com",
  "Providers": [
    {
      // Provider 1
    },
    {
      // Provider 2
    },
    {
      ...
    }
  ]
},
```

#### AWS

```json
{
  "Name": "aws",
  "AccessKey": "aws_access_key_id",
  "SecretKey": "aws_secret_key",
  "Region": "us-west-2"
}
```

#### File

```json
{
  "Name": "file",
  "Path": "/path/on/your/machine" //(optional)
}
```

#### SendGrid

```json
{
  "Name": "sendgrid",
  "ApiKey": "sendgrid_api_key"
}
```

#### SMTP

```json
{
  "Name": "smtp",
  "Host": "smtp.example.com",
  "Port": 123,
  "Username": "johndoe",
  "Password": "YourPassword123!",
  "StartTls": true/false,
  "Ssl": true/false,
  "SslOverride": true/false,
  "TrustServer": true/false // skips SSL certificate validation when set to `true`.
}
```

使用 SendGrid 的示例：

```json
{
  "Name": "smtp",
  "Host": "smtp.sendgrid.net",
  "Port": 465,
  "Username": "apikey",
  "Password": "<your-sendgrid-api-key>",
  "StartTls": false,
  "Ssl": true,
  "SslOverride": false,
  "TrustServer": true
}
```

### 环境变量 <a href="#environment-variables" id="environment-variables"></a>

参考：[ASP.NET Core 中的配置](https://learn.microsoft.com/zh-cn/aspnet/core/fundamentals/configuration/)

重要的是按照以下方式配置 'Mail\_\_From'。这是为了有一个备用的电子邮件地址来发送电子邮件。在 'Mail\_\_Providers' 中有一个数组，它是一个按顺序排列的电子邮件提供程序列表，如果它们失败，我们将按顺序尝试执行。要配置电子邮件提供程序，请参阅下面的子部分。

数组从零开始，因此我们配置 AWS 作为首先尝试发送电子邮件的选项，如果失败，我们将回退到 SendGrid。

```sh
-e Mail__From='johndoe@example.com'
-e Mail__FromName='John Doe'
-e Mail__Providers__0__Name='aws'
-e Mail__Providers__0__AccessKey='aws_access_key_id'
-e Mail__Providers__0__SecretKey='aws_secret_key'
-e Mail__Providers__1__Name='sendgrid'
-e Mail__Providers__1__ApiKey='sendgrid_api_key'
```

在下面的供应程序特定示例中，我们始终使用 '#' 作为提供程序索引。

#### AWS

```sh
-e Mail__Providers__#__Name='aws'
-e Mail__Providers__#__AccessKey='aws_access_key_id'
-e Mail__Providers__#__SecretKey='aws_secret_key'
-e Mail__Providers__#__Region='us-west-2'
```

#### File

```sh
-e Mail__Providers__#__Name='file'
-e Mail__Providers__#__Path='/path/on/your/machine' #(optional)
```

#### SendGrid

```sh
-e Mail__Providers__#__Name='sendgrid'
-e Mail__Providers__#__ApiKey='sendgrid_api_key'
```

#### SMTP

```sh
-e Mail__Providers__#__Name='smtp'
-e Mail__Providers__#__Host='smtp.example.com'
-e Mail__Providers__#__Port=123
-e Mail__Providers__#__Username='johndoe'
-e Mail__Providers__#__Password='YourPassword123!'
-e Mail__Providers__#__StartTls=true/false
-e Mail__Providers__#__Ssl=true/false
-e Mail__Providers__#__SslOverride=true/false
-e Mail__Providers__#__TrustServer=true/false # skips SSL certificate validation when set to `true`.
```

示例使用 SendGrid：

```sh
-e Mail__Providers__#__Name='smtp'
-e Mail__Providers__#__Host='smtp.sendgrid.net'
-e Mail__Providers__#__Port=465
-e Mail__Providers__#__Username='apikey'
-e Mail__Providers__#__Password='<your-sendgrid-api-key>'
-e Mail__Providers__#__StartTls=false
-e Mail__Providers__#__Ssl=true
-e Mail__Providers__#__SslOverride=false
-e Mail__Providers__#__TrustServer=true
```

## config.json

{% hint style="warning" %}
要求：

* 持久化存储，请参阅 “卷”。
{% endhint %}

`/etc/bitwarden_passwordless/config.json` 仅在您未指定以下环境变量时生成：

如果您将 `/etc/bitwarden_passwordless` 挂载到您的宿主机上。您可以为 `config.json` 指定配置。

如果以下键不存在，它们将被自动生成：

* Passwordless::ApiKey
* Passwordless::ApiSecret
* PasswordlessManagement::ManagementKey
* SALT\_KEY

建议您在第一次运行 `bitwarden/passwordless-self-host` 时自动生成它们。
