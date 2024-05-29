# =Android

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/frontend/android.html)
{% endhint %}

Passwordless.dev Android 客户端 SDK 使用户能够利用设备的内置指纹传感器和/或 FIDO 安全密钥，对支持 FIDO2 协议的网站和本机应用程序进行安全无密码访问。

## 要求 <a href="#requirements" id="requirements"></a>

* Android 9.0（API 级别 28）或更高版本
* Java 8 或更高版本
* [已完成「入门」指南](../get-started.md)

## 安装 <a href="#installation" id="installation"></a>

{% tabs %}
{% tab title="Apache Maven" %}
```xml
<dependency>
  <groupId>com.bitwarden</groupId>
  <artifactId>passwordless-android</artifactId>
  <version>1.0.1</version>
</dependency>
```
{% endtab %}

{% tab title="Gradle Kotlin DSL" %}
```kotlin
implementation("com.bitwarden:passwordless-android:1.0.1")
```
{% endtab %}

{% tab title="Gradle Grovy DSL" %}
```groovy
implementation 'com.bitwarden:passwordless-android:1.0.1'
```
{% endtab %}
{% endtabs %}

## 权限 <a href="#permissions" id="permissions"></a>

在您的 `AndroidManifest.xml` 中，添加以下权限：

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## 配置（Android 应用程序） <a href="#configuration-android-application" id="configuration-android-application"></a>

在 Android 应用程序的某个位置，配置 `PasswordlessOptions` 。

```kotlin
data class PasswordlessOptions(
   // Your public API key
   val apiKey: String,

   // Identifier for your server, for example 'example.com' if your backend is hosted at https://example.com.
   val rpId: String,

   // This is where your Facet ID goes
   val origin: String,

   // Where your backend is hosted
   val backendUrl:String,

   // Passwordless.dev server, change for self-hosting
   val apiUrl: String = "https://v4.passwordless.dev"
)
```

### .well-known/assetlinks.json <a href="#well-known-assetlinks-json" id="well-known-assetlinks-json"></a>

在应用程序的 `AndroidManifest.xml` 中，在 `manifest::application` 下添加以下标记：

```xml
<meta-data
  android:name="asset_statements"
  android:resource="@xml/assetlinks" />
```

在应用程序的 `res/xml/assetlinks.xml` 中，您需要添加以下内容。这将告诉我们的 Android 应用程序您的 `.well-known/assetlinks.json` 文件的托管位置。

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="assetlinks">https://yourexample.com/.well-known/assetlinks.json</string>
</resources>
```

### Facet ID <a href="#facet-id" id="facet-id"></a>

`Facet ID` 将在本指南后面用作 `origin` 。

要获取 Facet ID，请继续以下步骤。Facet ID 通常如下所示：`android:apk-key-hash:POIplOLeHuvl-XAQckH0DwY4Yb1ydnnKcmhn-jibZbk`

1、在终端中执行以下命令：

{% tabs %}
{% tab title="Bash" %}
```sh
# Linux, Mac OS, Git Bash, ...
keytool -list -v -keystore ~/.android/debug.keystore | grep "SHA256: " | cut -d " " -f 3 | xxd -r -p | basenc --base64url | sed 's/=//g'
```
{% endtab %}

{% tab title="Powershell" %}
```powershell
# Run keytool command and extract SHA256 hash
$keytoolOutput = keytool -list -v -keystore $HOME\.android\debug.keystore
$sha256Hash = ($keytoolOutput | Select-String "SHA256: ").ToString().Split(" ")[2]

# Remove any non-hex characters from the hash
$hexHash = $sha256Hash -replace "[^0-9A-Fa-f]"

# Convert the hexadecimal string to a byte array
$byteArray = [byte[]]@()
for ($i = 0; $i -lt $hexHash.Length; $i += 2) {
  $byteArray += [byte]([Convert]::ToUInt32($hexHash.Substring($i, 2), 16))
}

# Convert the byte array to a base64 string
$base64String = [Convert]::ToBase64String($byteArray)

# Convert the base64 string to a base64url string
$base64urlString = $base64String -replace '\+', '-' -replace '/', '_' -replace '=+$', ''

Write-Output $base64urlString
```
{% endtab %}
{% endtabs %}

2、调试密钥库的默认密码是 `android` 。对于您的生产密钥库，输入您选择的密码。

3、现在将结果附加到 `android:apk-key-hash:` 以获取 Facet ID：\
`android:apk-key-hash:POIplOLeHuvl-XAQckH0DwY4Yb1ydnnKcmhn-jibZbk`

## 配置（您的后端） <a href="#configuration-your-back-end" id="configuration-your-back-end"></a>

### 获取 SHA-256 证书指纹 <a href="#obtaining-the-sha-256-certificate-fingerprints" id="obtaining-the-sha-256-certificate-fingerprints"></a>

要配置后端，您需要在域的根目录托管一个 `.well-known/assetlinks.json` 文件。此文件包含允许通过后端进行身份验证的 SHA-256 证书指纹列表。

以下命令将打印有关具有指定别名的密钥库条目的详细信息，包括有关证书、其有效性和其他详细信息的信息。它通常在 Android 开发中用于验证调试密钥库的属性以及开发期间用于签名应用程序的关联密钥。

{% tabs %}
{% tab title="Gradle" %}
从终端进入项目根目录并运行以下命令：

```gradle
./gradlew signingReport
```
{% endtab %}

{% tab title="Bash" %}
```bash
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```
{% endtab %}

{% tab title="Powershell" %}
```powershell
keytool -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
```
{% endtab %}
{% endtabs %}

将此 SHA256 与您的根 Android 包名称一起放入后端，以便在 `https://yourexample.com/.well-known/assetlinks.json` 处为您的应用生成 `assetlinks.json` 。

如果您使用 `example-nodejs-backend` ，则只需将这 2 个值放入您的 `.env` 文件中即可。

### 托管 \~/.wellknown/assetlinks.json <a href="#host-well-known-assetlinks-json" id="host-well-known-assetlinks-json"></a>

您需要将以下文件存储在 `https://<your-domain>/.well-known/assetlinks.json` 处。要生成 SHA256 哈希，请阅读代码片段下方的链接。

您可能还需要更改 'target::namespace' 和 'target::package\_name' 属性以匹配您的应用程序。

```json
[
  {
    "relation": [
      "delegate_permission/common.handle_all_urls",
      "delegate_permission/common.get_login_creds"
    ],
    "target": {
      "namespace": "web"
    }
  },
  {
    "relation": [
      "delegate_permission/common.handle_all_urls",
      "delegate_permission/common.get_login_creds"
    ],
    "target": {
      "namespace": "android_app",
      "package_name": "com.example.myapplication",
      "sha256_cert_fingerprints": [
        "3C:E2:29:94:E2:DE:1E:EB:E5:F9:70:10:72:41:F4:0F:06:38:61:BD:72:76:79:CA:72:68:67:FA:38:9B:65:B9"
      ]
    }
  }
]
```

## 使用 PasswordlessClient <a href="#using-the-passwordlessclient" id="using-the-passwordlessclient"></a>

## 注册 <a href="#registration" id="registration"></a>

## 登录 <a href="#logging-in" id="logging-in"></a>

## 示例 <a href="#sample" id="sample"></a>

## 参考 <a href="#references" id="references"></a>
