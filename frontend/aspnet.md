# ASP.NET

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/frontend/aspnet.html)
{% endhint %}

## 入门 <a href="#getting-started" id="getting-started"></a>

使用我们的 NuGet 包将简化与 ASP.NET Identity 的集成过程，带来更顺畅、更高效的集成体验。

```sh
dotnet add package Passwordless.AspNetCore
```

引导现在是一个简单的过程，其中将 Passwordless 添加到 `IdentityBuilder` 成为必要步骤。

下面的示例假设您的 `appsettings.json` 将在关键 `Passwordless` 下添加所有配置设置。

```csharp
builder.Services
    .AddIdentity<IdentityUser, IdentityRole>()
    .AddEntityFrameworkStores<PasswordlessContext>()
    .AddPasswordless(builder.Configuration.GetSection("Passwordless"));
```

您至少需要提供 `ApiKey`（公钥）和 `ApiSecret`（私钥）。

```json
{
  "Passwordless": {
    "ApiKey": "***:public:***",
    "ApiSecret": "***:secret:***",
    "Register": {
      "Discoverable": true
    }
  }
}
```

现在，我们需要修改 `WebApplication` 对象以添加 Passwordless 路由，以便更轻松地注册、登录和管理凭据。

端点记录在本页后面的部分中。

```csharp
app.MapPasswordless(enableRegisterEndpoint: true);
```

现在我们将在 `/Account/Register` 创建注册页面。当我们加载页面时，我们将向用户呈现一个表单。

当我们单击 `Register` 按钮时，将调用 `OnPostAsync`。当表单有效时，我们将设置标志，该标志将允许我们的 Javascript 代码运行以允许注册令牌。

调用 `POST /passwordless-register` 将创建我们的 `IdentityUser` 并在其响应中返回注册令牌。然后我们将能够使用该令牌来创建我们的通行密钥。

```html
@page

@using Passwordless.Net
@using Microsoft.Extensions.Options
@using Passwordless.AspNetCore

@model RegisterModel

@inject IOptions<PasswordlessAspNetCoreOptions> PasswordlessOptions;

@{
    ViewData["Title"] = "Register";
}
<h1>@ViewData["Title"]</h1>

@{
    var canAddPasskeys = ViewData["CanAddPasskeys"] is true;
}

<div class="row">
    <div class="col-12">
        <form class="needs-validation" action="" method="POST">
            <div class="mb-3">
                <label asp-for="Form.Username" class="form-label">Username</label>
                <input placeholder="Jane Doe" type="text" asp-for="Form.Username" class="form-control" id="username">
                <span class="text-danger" asp-validation-for="Form.Username"></span>
            </div>
            <div class="mb-3">
                <label asp-for="Form.Email" class="form-label">Email</label>
                <input placeholder="janedoe@example.org" type="text" asp-for="Form.Email" class="form-control" id="email">
                <span class="text-danger" asp-validation-for="Form.Email"></span>
            </div>
            <div class="text-danger" asp-validation-summary="ModelOnly"></div>
            <div>
                <button type="submit" class="btn-primary">Register</button>
            </div>
        </form>
    </div>
</div>

@if (canAddPasskeys)
{
    <script src="https://cdn.passwordless.dev/dist/1.1.0/umd/passwordless.umd.js"></script>
    <script>
        async function register() {
            const username = document.getElementById("username").value;
            const email = document.getElementById("email").value;
            const registrationRequest = {
                email: email,
                username: username,
                displayName: username,
                aliases: [email]
            };
        
            const registrationResponse = await fetch('/passwordless-register', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(registrationRequest)
            });
        
            // 如果没有错误，则进行反序列化，然后使用返回的标记创建我们的通行密钥
            if (registrationResponse.ok) {
                const registrationResponseJson = await registrationResponse.json();
                const token = registrationResponseJson.token;
        
                // 我们需要使用从 https://cdn.passwordless.dev/dist/1.1.0/umd/passwordless.umd.js 导入的客户端。
                const p = new Passwordless.Client({
                    apiKey: "@PasswordlessOptions.Value.ApiKey",
                    apiUrl: "@PasswordlessOptions.Value.ApiUrl"
                });
                const registeredPasskeyResponse = await p.register(token, email);
            }
        }
        
        register();
    </script>
}
```

```csharp
public IActionResult OnGet()
{
    if (HttpContext.User.Identity is { IsAuthenticated: true })
    {
        return LocalRedirect("/");
    }

    return Page();
}

public async Task OnPostAsync(FormModel form, CancellationToken cancellationToken)
{
    if (!ModelState.IsValid) return;
    _logger.LogInformation("Registering user {username}", form.Username);
    ViewData["CanAddPasskeys"] = true;
}

public FormModel Form { get; init; } = new();
```

关于登录页面 `/Account/Login`，单击 'Login' 按钮将再次触发一个类似的标志设置，从而执行我们的 JavaScript 代码的执行。

您可以采取两种方法。您可以请求一个 "alias" 来验证哪些通行密钥与别名对应，或者如果您有可发现的凭据，可能就不需要输入表单了。

登录过程与注册过程不同。在登录过程中，您将向身份验证器请求有效的密钥，以获取一个验证令牌。随后，该验证令牌将被传输到您的后端以启动会话。

使用 "Passwordless ASP.NET Identity SDK"，您可以通过简单调用 POST /passwordless-login 来简化此过程，并且 SDK 将为您处理所有必要的步骤。

身份验证成功后，我们的示例应用程序将自动将您重定向到 /Authorized/HelloWorld 页面，该页面需要您登录才能访问。

```html
@page
@model LoginModel
@using Microsoft.Extensions.Options
@using Passwordless.AspNetCore
@inject IOptions<PasswordlessAspNetCoreOptions> PasswordlessOptions;

@{
    ViewData["Title"] = "Login";
}
<h1>@ViewData["Title"]</h1>

@{
    var canLogin = ViewData["CanLogin"] != null && (bool)ViewData["CanLogin"];
}

<div class="row">
    <div class="col-12">
        <form class="needs-validation" action="" method="POST">
            <div class="mb-3">
                <label asp-for="Form.Email" class="form-label">Email</label>
                <input placeholder="janedoe@example.org" type="text" asp-for="Form.Email" class="form-control" id="email">
                <span class="text-danger" asp-validation-for="Form.Email"></span>
            </div>
            <div class="text-danger" asp-validation-summary="ModelOnly"></div>
            <div>
                <button type="submit" class="btn-primary">Login</button>
            </div>
        </form>
    </div>
</div>

@if (canLogin)
{
    <script src="https://cdn.passwordless.dev/dist/1.1.0/umd/passwordless.umd.js"></script>
    <script>
        async function login() {
            const alias = document.getElementById("email").value;
            const p = new Passwordless.Client({
                apiKey: "@PasswordlessOptions.Value.ApiKey",
                apiUrl: "@PasswordlessOptions.Value.ApiUrl"
            });
            const loginPasskeyResponse = await p.signinWithAlias(alias);
            if (!loginPasskeyResponse) {
                return;
            }
            const loginRequest = {
                token: loginPasskeyResponse.token
            };
            const loginResponse = await fetch('/passwordless-login', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(loginRequest)
            });
        
            if (loginResponse.ok) {
                console.log('login successful: ' + await loginResponse.text());
        
                // 重定向到授权页面 /Authorized/HelloWorld
                window.location.href = '/Authorized/HelloWorld';
            }
        }
        
        login();
    </script>
}
```

如果我们在已通过身份验证的情况下访问登录页面，我们希望重定向到其他地方。

```csharp
public IActionResult OnGet()
{
    if (HttpContext.User.Identity is { IsAuthenticated: true })
    {
        return LocalRedirect("/");
    }
    return Page();
}

public async Task OnPostAsync(LoginForm form, CancellationToken cancellationToken)
{
    if (!ModelState.IsValid) return;
    _logger.LogInformation("Logging in user {email}", form.Email);
    ViewData["CanLogin"] = true;
}

public LoginForm Form { get; } = new();
```

然后，您可以从 `HttpContext` 访问有关登录用户的信息：

```csharp
public class HelloWorldModel : PageModel
{
    private readonly ILogger<HelloWorldModel> _logger;
    
    public HelloWorldModel(ILogger<HelloWorldModel> logger)
    {
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    public void OnGet()
    {
        var identity = HttpContext.User.Identity ;
        var email = User.FindFirstValue(ClaimTypes.Email);
        AuthenticatedUser = new AuthenticatedUserModel(identity.Name, email);
    }
    
    public AuthenticatedUserModel AuthenticatedUser { get; private set; }
}

public record AuthenticatedUserModel(string Username, string Email);
```

## 配置 <a href="#configuration" id="configuration"></a>

| 密钥                             | 默认值                         | 描述                                                                                                                                                                               |
| ------------------------------ | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ApiKey                         |                             | 您的公共 API 密钥。                                                                                                                                                                     |
| ApiSecret                      |                             | 您的私有 API 密钥。                                                                                                                                                                     |
| ApiUrl                         | https://v4.passwordless.dev | 您的 `Passwordless.dev` 实例运行的位置。                                                                                                                                                   |
| SignInScheme                   | Identity.Application        | 控制用于处理登录的方案。                                                                                                                                                                     |
| UserIdClaimType                |                             | 控制用于从已经过身份验证的用户查找用户 ID 的声明类型，请参阅 `ClaimsPrincipal`。如果为空，则将从 `IdentityOptions.ClaimsIdentity` 回退到 `ClaimsIdentityOptions.UserIdClaimType`；如果又为空，则回退到 `ClaimTypes.NameIdentifier`。 |
| Register\_\_AliasHashing       | true                        | \[false/true] 当设置为 t`rue` 时，别名仅以其哈希形式存储。                                                                                                                                         |
| Register\_\_Attestation        | None                        | \[None/Direct/Indirect] 值为 `None` 表示服务器不关心验证。值为 `Indirect` 表示服务器将允许匿名验证数据。`Direct` 表示服务器希望接收来自身份验证器的验证数据。                                                                        |
| Register\_\_AuthenticationType | any                         |                                                                                                                                                                                  |
| Register\_\_Expiration         | "00:02:00"                  | \["hh:mm:ss"]                                                                                                                                                                    |
| Register\_\_Discoverable       | true                        | \[false/true] 可发现凭证将私钥和关相关数据存储在身份验证器的内存中，而不是服务器上。这样，服务器就无需将证书发送到验证器进行解密，从而减少了身份验证对用户名和密码的依赖。                                                                                     |
| Register\_\_UserVerification   | Preferred                   | \[Discouraged/Preferred/Required] WebAuthn 依赖方可能在某些操作中需要用户验证，而在其他操作中则不需要，因此可以使用这种类型来表达自己的需求。                                                                                     |

## 高级 <a href="#advanced" id="advanced"></a>

如果您需要更大的灵活性，我们邀请您探索我们的 JavaScript 客户端库和 .NET SDK。如果您希望对 ASP.NET Identity 框架进行更精细的控制，或者希望进行完全自定义的实施，则此选项特别合适。

* [JavaScript 客户端](javascript.md)
* [.NET](../backend/dotnet.md)
