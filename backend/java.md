# Java

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/backend/java.html)
{% endhint %}

## 入门 <a href="#getting-started" id="getting-started"></a>

1、将我们的包的依赖项添加到您的 `pom.xml` 中：

```xml
<dependency>
    <groupId>com.bitwarden</groupId>
    <artifactId>passwordless</artifactId>
    <version>1.0.5</version>
</dependency>
```

## 示例 <a href="#example" id="example"></a>

此 Java 实现与 Java 8 及更高版本兼容。[注册](../api.md#register-token)函数可能看起来像这样：

### 创建 `PasswordlessClient` 实例： <a href="#create-passwordlessclient-instance" id="create-passwordlessclient-instance"></a>

```java
import com.bitwarden.passwordless.*;

import java.io.*;

public class PasswordlessJavaSdkExample implements Closeable {

    private final PasswordlessClient client;

    public PasswordlessClientExample() {
        PasswordlessOptions options = PasswordlessOptions.builder()
                .apiSecret("your_api_secret")
                .build();

        client = PasswordlessClientBuilder.create(options)
                .build();
    }

    @Override
    public void close() throws IOException {
        client.close();
    }
}
```

**注意**：使用带有 `close` 方法的 `PasswordlessClient` 后，需要关闭底层 http 客户端资源。

### 注册通行密钥 <a href="#register-a-passkey" id="register-a-passkey"></a>

```java
import com.bitwarden.passwordless.*;
import com.bitwarden.passwordless.error.*;
import com.bitwarden.passwordless.model.*;

import java.io.*;
import java.util.*;

public class PasswordlessJavaSdkExample {

    private final PasswordlessClient client;

    // 构造函数

    public String getRegisterToken(String alias) throws PasswordlessApiException, IOException {

        // 从会话中获取现有用户 ID 或创建新用户
        String userId = UUID.randomUUID().toString();

        // 提供给 Api 的选项
        RegisterToken registerToken = RegisterToken.builder()
                // 您的用户 id
                .userId(userId)
                // 例如用户电子邮件地址，将显示在浏览器 UI 中
                .username(alias)
                // 可选：将此用户 ID 链接到别名（例如电子邮件地址）
                .aliases(Arrays.asList(alias))
                .build();

        RegisteredToken response = client.registerToken(registerToken);

        // 返回此令牌
        return response.getToken();
    }
}
```

### 验证用户 <a href="#verify-user" id="verify-user"></a>

```java
import com.bitwarden.passwordless.*;
import com.bitwarden.passwordless.error.*;
import com.bitwarden.passwordless.model.*;

import java.io.*;

public class PasswordlessJavaSdkExample {

    private final PasswordlessClient client;

    // Constructor

    public VerifiedUser verifySignInToken(String token) throws PasswordlessApiException, IOException {

        VerifySignIn signInVerify = VerifySignIn.builder()
                .token(token)
                .build();

        // 用户登录、设置 cookie 等
        return client.signIn(signInVerify);
    }
}
```

### 自定义 <a href="#customization" id="customization"></a>

通过向 `apiSecret` 提供您的应用程序的私有 API 密钥来自定义 `PasswordlessOptions`。如果您喜欢自行托管，也可以更改 `apiUrl`。

通过提供 `httpClient` [CloseableHttpClient](https://hc.apache.org/httpcomponents-client-5.2.x/index.html) 实例和 `objectMapper` [ObjectMapper](https://github.com/FasterXML/jackson-databind) 来自定义`PasswordlessClientBuilder`。

### 示例 <a href="#examples" id="examples"></a>

有关使用此库的 Spring Boot 3 应用程序，请参阅 [Passwordless Java 示例](https://github.com/passwordless/passwordless-java/tree/main/examples/spring-boot-3-jdk-17)。
