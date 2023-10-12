# Java 1.8+

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/backend/java.html)
{% endhint %}

## 入门 <a href="#getting-started" id="getting-started"></a>

1、将托管我们的包的存储库添加到您的 `pom.xml` 中：

```xml
<repositories>
    <repository>
        <id>ossrh</id>
        <url>https://s01.oss.sonatype.org/content/repositories/releases</url>
    </repository>
</repositories>
```

2、将我们的包的依赖项添加到您的 `pom.xml` 中：

```xml
<dependency>
    <groupId>com.bitwarden</groupId>
    <artifactId>passwordless</artifactId>
    <version>1.0.5</version>
</dependency>
```

## 示例 <a href="#example" id="example"></a>

此 Java 实现与 Java 1.8 及更高版本兼容。[注册](../api.md#register-token)函数可能看起来像这样：

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

    // Constructor

    public String getRegisterToken(String alias) throws PasswordlessApiException, IOException {

        // Get existing userid from session or create a new user.
        String userId = UUID.randomUUID().toString();

        // Options to give the Api
        RegisterToken registerToken = RegisterToken.builder()
                // your user id
                .userId(userId)
                // e.g. user email, is shown in browser ui
                .username(alias)
                // Optional: Link this userid to an alias (e.g. email)
                .aliases(Arrays.asList(alias))
                .build();

        RegisteredToken response = client.registerToken(registerToken);

        // return this token
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

        // Sign the user in, set a cookie, etc,
        return client.signIn(signInVerify);
    }
}
```

### 自定义 <a href="#customization" id="customization"></a>

通过向 `apiSecret` 提供您的应用程序的私有 API 密钥来自定义 `PasswordlessOptions`。如果您喜欢自行托管，也可以更改 `apiUrl`。

通过提供 `httpClient` [CloseableHttpClient](https://hc.apache.org/httpcomponents-client-5.2.x/index.html) 实例和 `objectMapper` [ObjectMapper](https://github.com/FasterXML/jackson-databind) 来自定义`PasswordlessClientBuilder`。

### 示例 <a href="#examples" id="examples"></a>

有关使用此库的 Spring Boot 3 应用程序，请参阅 [Passwordless Java 示例](https://github.com/passwordless/passwordless-java-example)。
