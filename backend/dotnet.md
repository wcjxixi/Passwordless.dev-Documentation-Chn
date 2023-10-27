# .NET

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/backend/dotnet.html)
{% endhint %}

安装 NuGet 包：

```sh
dotnet add package Passwordless
```

此 ASP.NET Core 实现使用 .NET 7 和某些 JavaScript 来实现简单的 passwordless.dev。[注册](../api.md#register-token)函数可能看起来像这样：

`Startup` 类负责配置和设置应用程序在其生命周期中将使用的各种服务、中间件和组件。

```javascript
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }
    
    private IConfiguration Configuration { get; }

    ...

    // 该方法由 runtime 调用。使用此方法向容器添加服务。
    // 有关如何配置应用程序的详细信息，请访问 https://go.microsoft.com/fwlink/?LinkID=398940
    public void ConfigureServices(IServiceCollection services)
    {
        // 添加对路由到控制器的支持
        services.AddControllers();
        
        // 注入 Passwordless SDK
        services.Configure<PasswordlessOptions>(Configuration.GetRequiredSection("Passwordless"));
        services.AddPasswordlessSdk(options =>
        {
            Configuration.GetRequiredSection("Passwordless").Bind(options);
        });
    }
```

较新的模板可能没有此功能，您需要改为在 `Program` 类中配置此功能。在这种情况下，您可以访问 `WebApplicationBuilder` 上的 `Services` 和 `Configuration` 属性。

将 `IPasswordlessClient` 注入到您想要使用它的类中就很简单了。使用 SDK 注册令牌看起来像这样：

```javascript
[HttpGet("/create-token")]
public async Task<IActionResult> GetRegisterToken(string alias)
{
    var userId = Guid.NewGuid().ToString();

    var payload = new RegisterOptions(userId, alias)
    {
        Aliases = new HashSet<string> { alias }
    };

    try
    {
        var token = await _passwordlessClient.CreateRegisterTokenAsync(payload);
        return Ok(token);
    }
    catch (PasswordlessApiException e)
    {
        return new JsonResult(e.Details)
        {
            StatusCode = (int?)e.StatusCode,
        };
    }
}
```

登录看起来像这样：

```javascript
[HttpGet("/verify-signin")]
public async Task<IActionResult> VerifySignInToken(string token)
{
    try
    {
        var verifiedUser = await _passwordlessClient.VerifyTokenAsync(token);
        return Ok(verifiedUser);
    }
    catch (PasswordlessApiException e)
    {
        return new JsonResult(e.Details)
        {
            StatusCode = (int?)e.StatusCode
        };
    }
}
```

## 参考 <a href="#references" id="references"></a>

* [Github 上的 ASP.NET 示例](https://github.com/passwordless/passwordless-dotnet/tree/main/examples/Passwordless.Example)
