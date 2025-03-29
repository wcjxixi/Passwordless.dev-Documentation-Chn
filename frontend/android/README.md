# Android

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/frontend/android.html)
{% endhint %}

Passwordless.dev Android 客户端 SDK 使用户能够利用设备的内置指纹传感器和/或 FIDO 安全密钥，对支持 FIDO2 协议的网站和本机应用程序进行安全的无密码访问。

## 要求 <a href="#requirements" id="requirements"></a>

* Android 9.0（API 级别 28）或更高版本
* Java 8 或更高版本
* [已完成「入门」指南](../../get-started.md)

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



{% tabs %}
{% tab title="使用 Dagger Hilt" %}
您可以通过注入 Dagger Hilt 来设置 `ActivityContext` 和 `CoroutineScope`，方法如下：

```kotlin
@Module
@InstallIn(ActivityComponent::class)
class PasswordlessModule {
    @Provides
    fun provideLifecycleCoroutineScope(activity: Activity): LifecycleCoroutineScope =
        (activity as AppCompatActivity).lifecycleScope

    @Provides
    @ActivityScoped
    fun providePasswordlessClient(
        @ActivityContext activity: Context, scope: LifecycleCoroutineScope): PasswordlessClient {
        val options = PasswordlessOptions(
            DemoPasswordlessOptions.API_KEY,
            DemoPasswordlessOptions.RP_ID,
            DemoPasswordlessOptions.ORIGIN,
            DemoPasswordlessOptions.API_URL
        )

        return PasswordlessClient(options, activity, scope)
    }
}
```
{% endtab %}

{% tab title="不使用 Dagger Hilt" %}
或者也可以手动设置无密码客户端的上下文。确保上下文设置为当前 `Activity`。

<pre class="language-kotlin"><code class="lang-kotlin">/** 需要根据当前活动设置上下文
 * 如果有不同的活动处理寄存器和签名，
<strong> * 然后在每个活动中调用它
</strong>*/
_passwordless.setContext(this)
</code></pre>

设置 Coroutine 范围，传递当前片段的 lifecycleScope，只有在您再次不使用 Dagger Hilt 的情况下才有必要。

```kotlin
_passwordless.setCoroutineScope(lifecycleScope)
```
{% endtab %}
{% endtabs %}

## 注册 <a href="#registration" id="registration"></a>

1. 用户界面 (UI)：
   * （必填）指定凭据的用户名。
   * （可选）如果是不可发现的凭据，则将用户作为别名输入。
2. 调用 **`your backend`** 实现的 `POST /register/token`：您的后端应调用 Passwordless.de API 生成注册令牌。
   * 参阅 [API 文档](../../api.md#register-token)
3. 调用 `PasswordlessClient.register()`：

```kotlin
// 将注册令牌传递给 PasswordlessClient.register()
_passwordless.register(
    token = responseToken,
    nickname = nickname
) { success, exception, result ->
    if (success) {
        Toast.makeText(context, result.toString(), Toast.LENGTH_SHORT).show()
    } else {
        Toast.makeText(context, "Exception: " + getPasskeyFailureMessage(exception as Exception), Toast.LENGTH_SHORT).show()
    }
}
```



## 登录 <a href="#logging-in" id="logging-in"></a>

1. 用户界面 (UI)：（可选）如果是不可发现的凭据，则将用户作为别名输入。
2. 调用 `PasswordlessClient.login()`：使用（可选）别名和响应回调启动登录过程。
3. 调用 `your backend` 实现的 `POST /register/token`：您的后端应调用 Passwordless.de API 生成注册令牌。
   * 参阅 [API 文档](../../api.md#register-token)

```kotlin
// 使用别名调用 PasswordlessClient.login()
_passwordless.login(alias) { success, exception, result ->
    if (success) {
        lifecycleScope.launch {
            // 成功后，调用后端验证令牌，例如返回一个 JWT 令牌。
            val clientDataResponse = httpClient.login(UserLoginRequest(result?.token!!))
            if (clientDataResponse.isSuccessful) {
                // 登录成功。 解析响应或 JWT 令牌并继续。
                val data = clientDataResponse.body()
                showText(data.toString())
            }
        }
    } else {
        showException(exception)
    }
}
```

## 示例 <a href="#sample" id="sample"></a>

您可以在[这里](https://github.com/bitwarden/passwordless-android/tree/main/app)找到示例应用程序的源代码。

{% tabs %}
{% tab title="注册 1" %}
{% embed url="https://docs.passwordless.dev/assets/register_1-CtQIHsm_.png" %}
{% endtab %}

{% tab title="注册 2" %}
{% embed url="https://docs.passwordless.dev/assets/register_2-CNsiRMHG.png" %}
{% endtab %}

{% tab title="登录 1" %}
{% embed url="https://docs.passwordless.dev/assets/login_1-1GH5b4rj.png" %}
{% endtab %}

{% tab title="登录 2" %}
{% embed url="https://docs.passwordless.dev/assets/login_2-D_RgitXZ.png" %}
{% endtab %}

{% tab title="登录 3" %}
{% embed url="https://docs.passwordless.dev/assets/login_3-Cdru5hSl.png" %}
{% endtab %}
{% endtabs %}

## 参考 <a href="#references" id="references"></a>

* [入门 - Passwordless.dev](../../get-started.md)
* [使用您的后端集成 - Passwordless.dev](../../backend/)
* [客户端身份验证 - Google](https://developers.google.com/android/guides/client-auth?hl=zh-cn)
* [关联应用程序和网站 - Google](https://developers.google.com/identity/smartlock-passwords/android/associate-apps-and-sites?hl=zh-cn)
* [通行密匙 - Google](https://developer.android.com/identity/sign-in/credential-manager?hl=zh-cn)
* [故障排除 - Android 客户端 SDK - Passwordless.dev](troubleshooting.md)
