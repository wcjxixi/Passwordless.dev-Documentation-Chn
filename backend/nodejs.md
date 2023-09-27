# Node.js

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/backend/nodejs.html)
{% endhint %}

## 安装 <a href="#installation" id="installation"></a>

```sh
$ npm i @@passwordlessdev/passwordless-nodejs
```

## 要求 <a href="#requirements" id="requirements"></a>

* ES2018 或更高版本，[此处](https://node.green/)了解更多信息。
* 支持的 JavaScript 模块：CommonJS、ES
* Node.js 10 或更高版本

## 使用 <a href="#using" id="using"></a>

指定 `baseUrl` 是可选的，并将包含 `https://v4.passwordless.dev` 作为其默认值。

要获取您的 `ApiSecret`，您可以在管理控制台的 `Applications > 'Your Application' > Getting Started` 下找到它。

```javascript
const options: PasswordlessOptions = {
  baseUrl: 'https://v4.passwordless.dev'
};
this._passwordlessClient = new PasswordlessClient('demo:secret:f831e39c29e64b77aba547478a4b3ec6', options);
```

```javascript
const options: PasswordlessOptions = {};
this._passwordlessClient = new PasswordlessClient('demo:secret:f831e39c29e64b77aba547478a4b3ec6', options);
```

### 注册 <a href="#registration" id="registration"></a>

例如，如果您有一个带有「signup」箭头功能的「UserController.ts」。您可以如下所示注册一个新的令牌。

在注册令牌之前，您首先需要将新用户存储在数据库中并验证其是否成功。

```javascript
signup = async (request: express.Request, response: express.Response) => {
        const signupRequest: SignupRequest = request.body;
        const repository: UserRepository = new UserRepository();
        let id: string = null;
        try {
            // 首先，在数据库中创建用户。我们将使用 id 来注册令牌。
            // 这样我们就知道凭证属于哪个用户。
            id = repository.create(signupRequest.username, signupRequest.firstName, signupRequest.lastName);
        } catch {
            // 创建用户失败，进行错误处理。
        } finally {
            repository.close();
        }

        if (!id) {
            // 我们无法创建用户，因此不要继续创建令牌。
            response.send(400);
        }

        let registerOptions = new RegisterOptions();
        registerOptions.userId = id;
        registerOptions.username = signupRequest.username;
        
        // 我们将使用我们的 deviceName 作为别名。但使用别名是完全可选的。
        if (signupRequest.deviceName) {
            registerOptions.aliases = new Array(1);
            registerOptions.aliases[0] = signupRequest.deviceName;
        }
        
        registerOptions.discoverable = true;
        
        // 现在调用 Passwordless.dev 来注册一个新的令牌。
        const token: RegisterTokenResponse = await this._passwordlessClient.createRegisterToken(registerOptions);
        
        // 将令牌返回给客户端。
        response.send(token);
    }
```

### 登录 <a href="#logging-in" id="logging-in"></a>

```javascript
signin = async (request: express.Request, response: express.Response) => {
        try {
            const token: string = request.query.token as string;
            
            // 首先检查令牌是否有效，以及是否找到匹配的用户。
            const verifiedUser: VerifiedUser = await this._passwordlessClient.verifyToken(token);

            // 如果找到用户并且令牌有效，您可以继续登录该用户。
            if (verifiedUser && verifiedUser.success === true) {
                // 如果您想为在客户端呈现的 SPA 构建 JWT 令牌，您可以在此处执行此操作。
                response.send(JSON.stringify(verifiedUser));
                return;
            }
        } catch (error) {
            console.error(error.message);
        }
        response.send(401);
    }
```

## 参考 <a href="#references" id="references"></a>

* [Github 上的 Node.js 示例](https://github.com/passwordless/passwordless-nodejs-example)
* 如果您只想查看 Node.js 演示的运行，我们在 [demo-backend.passwordless.dev](https://demo-backend.passwordless.dev/) 上托管​​了一个示例
