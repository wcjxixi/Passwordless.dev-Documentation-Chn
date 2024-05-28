# 高级

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/self-hosting/advanced.html)
{% endhint %}

## 示例：使用 SQL Server <a href="#example-using-sql-server" id="example-using-sql-server"></a>

您的 docker-compose.yml 可能包含：

```yaml
---
version: '3.8'

services:
  bitwarden-passwordless:
    depends_on:
      - db
    environment:
      BWP_ENABLE_SSL: true
      BWP_PORT: 443
      BWP_DB_PROVIDER: 'mssql'
      BWP_DB_SERVER: 'localhost'
      BWP_DB_PORT: 1433
      BWP_DB_DATABASE_API: 'Api'
      BWP_DB_DATABASE_ADMIN: 'Admin'
      BWP_DB_USERNAME: 'sa'
      BWP_DB_PASSWORD: 'super_strong_password'
    image: ${REGISTRY:-bitwarden}/passwordless:latest
    restart: always
    ports:
      - '443:5701'
    volumes:
      - bitwarden_passwordless:/etc/bitwarden_passwordless
      - logs:/var/log/bitwarden_passwordless

  db:
    environment:
      MSSQL_SA_PASSWORD: 'super_strong_password'
      ACCEPT_EULA: Y
    image: mcr.microsoft.com/mssql/server:2022-latest
    restart: always
    volumes:
      - data:/var/opt/mssql

volumes:
  bitwarden_passwordless:
  logs:
  data:
```
