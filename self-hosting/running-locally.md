# 本地运行

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/self-hosting/running-locally.html)
{% endhint %}

## 简单示例 #1 <a href="#simple-example-1" id="simple-example-1"></a>

* 可通过 `https://localhost:5042/` 访问管理控制台
* 无永久存储：在此示例中，生成的 `config.json` 和 `Sqlite databases` 将丢失。

```sh
docker pull bitwarden/passwordless-self-host:stable
docker run \
  --publish 5042:5701 \
  --env BWP_ENABLE_SSL=true \
  bitwarden/passwordless-self-host:stable
```

## 简单示例 #2 <a href="#simple-example-2" id="simple-example-2"></a>

* 可通过 `http://localhost:5042/` 访问管理控制台
* 永久存储：在此示例中，生成的 `config.json` 和 `Sqlite databases` 将保留在主机上的 `/your/directory` 目录中。

```sh
docker pull bitwarden/passwordless-self-host:stable
docker run \
  --publish 5042:5701 \
  --volume /your/directory:/etc/bitwarden_passwordless \
  --env BWP_PORT=5042 \
  bitwarden/passwordless-self-host:stable
```

## SSL 示例 <a href="#example-with-ssl" id="example-with-ssl"></a>

* 可通过 `http://localhost:5042/` 访问管理控制台
* 永久存储：在此示例中，生成的 `config.json` 和 `Sqlite databases` 将保留在主机上的 `/your/directory` 目录中。

```sh
docker pull bitwarden/passwordless-self-host:stable
docker run \
  --publish 5042:5701 \
  --volume /your/directory:/etc/bitwarden_passwordless \
  --env BWP_PORT=5042 \
  --env BWP_ENABLE_SSL=true \
  bitwarden/passwordless-self-host:stable
```

{% hint style="warning" %}
执行 `docker run` 命令后，你会在主机系统的指定目录或容器中的 `/etc/bitwarden_passwordless` 路径下找到 SSL 证书。

为确保功能正常，必须在本地机器上注册 `ssl.crt` 证书。否则，网页浏览器可能会将 SSL 证书标记为无效，从而导致 WebAuthn 功能无法运行，因为 WebAuthn API 不会在不安全的环境中暴露。
{% endhint %}
