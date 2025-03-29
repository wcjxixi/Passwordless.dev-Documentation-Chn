# 故障排除

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/frontend/android/troubleshooting.html)
{% endhint %}

## 可在物理设备上运行，但不能在模拟器中运行 <a href="#working-on-physical-devices-but-not-in-the-emulator" id="working-on-physical-devices-but-not-in-the-emulator"></a>

此错误表示框架返回的错误。

```
2024-02-22 11:07:55.231  6884-6908  CredManProvService      dev.passwordless.sampleapp           I  CreateCredentialResponse error returned from framework
2024-02-22 11:07:55.238  6884-6979  bwp-register-begin      dev.passwordless.sampleapp           E  Cannot call registerRequest
    androidx.credentials.exceptions.publickeycredential.CreatePublicKeyCredentialDomException
        at androidx.credentials.exceptions.publickeycredential.DomExceptionUtils$Companion.generateException(DomExceptionUtils.kt:144)
        at androidx.credentials.exceptions.publickeycredential.DomExceptionUtils$Companion.access$generateException(DomExceptionUtils.kt:57)
        at androidx.credentials.exceptions.publickeycredential.CreatePublicKeyCredentialDomException$Companion.createFrom(CreatePublicKeyCredentialDomException.kt:127)
        at androidx.credentials.exceptions.publickeycredential.CreatePublicKeyCredentialException$Companion.createFrom(CreatePublicKeyCredentialException.kt:51)
        at androidx.credentials.CredentialProviderFrameworkImpl.convertToJetpackCreateException$credentials_release(CredentialProviderFrameworkImpl.kt:315)
        at androidx.credentials.CredentialProviderFrameworkImpl$onCreateCredential$outcome$1.onError(CredentialProviderFrameworkImpl.kt:201)
        at androidx.credentials.CredentialProviderFrameworkImpl$onCreateCredential$outcome$1.onError(CredentialProviderFrameworkImpl.kt:187)
        at android.credentials.CredentialManager$CreateCredentialTransport.lambda$onError$2(CredentialManager.java:752)
        at android.credentials.CredentialManager$CreateCredentialTransport.$r8$lambda$8NwBIrbcK6SvF9Mra_qL_8hhFMU(Unknown Source:0)
        at android.credentials.CredentialManager$CreateCredentialTransport$$ExternalSyntheticLambda0.run(Unknown Source:6)
        at androidx.credentials.CredentialManager$$ExternalSyntheticLambda0.execute(Unknown Source:0)
        at android.credentials.CredentialManager$CreateCredentialTransport.onError(CredentialManager.java:751)
        at android.credentials.ICreateCredentialCallback$Stub.onTransact(ICreateCredentialCallback.java:123)
        at android.os.Binder.execTransactInternal(Binder.java:1344)
        at android.os.Binder.execTransact(Binder.java:1275)
```

解决方案：

* 验证 Google Play 服务是否为最新版本，必要时更新 Google Play 服务。
