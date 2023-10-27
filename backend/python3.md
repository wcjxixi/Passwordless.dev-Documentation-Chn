# Python 3+

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/backend/python3.html)
{% endhint %}

## 入门 <a href="#getting-started" id="getting-started"></a>

使用 `python -m pip install passwordless` 安装。

## 示例 <a href="#example" id="example"></a>

此 Python 3 实现与 Python 3.8+ 及更高版本兼容。[注册](../api.md#register-token)函数可能看起来像这样：

### 创建 `PasswordlessClient` 实例： <a href="#create-passwordlessclient-instance" id="create-passwordlessclient-instance"></a>

```python
from passwordless import (
    PasswordlessClient,
    PasswordlessClientBuilder,
    PasswordlessOptions
)


class PasswordlessPythonSdkExample:
    client: PasswordlessClient

    def __init__(self):
        options = PasswordlessOptions("your_api_secret")

        self.client = PasswordlessClientBuilder(options).build()

```

### 注册通行密钥 <a href="#register-a-passkey" id="register-a-passkey"></a>

```python
import uuid
from passwordless import (
    PasswordlessClient,
    RegisterToken,
    RegisteredToken
)


class PasswordlessPythonSdkExample:
    client: PasswordlessClient

    def get_register_token(self, alias: str) -> str:
        # 从会话中获取现有用户 ID 或创建新用户
        user_id = str(uuid.uuid4())

        # 提供给 Api 的选项
        register_token = RegisterToken(
            user_id=user_id,  # 您的用户 id
            username=alias,  # 例如用户电子邮件地址，将显示在浏览器 UI 中
            aliases=[alias]  # 可选：将此用户 ID 链接到别名（例如电子邮件地址）
        )

        response: RegisteredToken = self.client.register_token(register_token)

        # 返回此令牌
        return response.token
```

### 验证用户 <a href="#verify-user" id="verify-user"></a>

```python
from passwordless import (
    PasswordlessClient,
    VerifySignIn,
    VerifiedUser
)


class PasswordlessPythonSdkExample:
    client: PasswordlessClient

    def verify_sign_in_token(self, token: str) -> VerifiedUser:
        verify_sign_in = VerifySignIn(token)

        # 用户登录、设置 cookie 等
        return self.client.sign_in(verify_sign_in)
```

### 自定义 <a href="#customization" id="customization"></a>

通过向 `api_secret` 提供您的应用程序的 Api Secret 来自定义 `PasswordlessOptions`。如果您喜欢自行托管，也可以更改 `api_url`。

通过提供 `session` 请求会话来自定义 `PasswordlessClientBuilder` 实例。

### 示例 <a href="#examples" id="examples"></a>

请参阅 Flash Web 应用程序的 [Passwordless Python 示例](https://github.com/passwordless/passwordless-python/tree/main/examples/flask)。
