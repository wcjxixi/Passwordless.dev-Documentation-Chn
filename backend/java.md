# Java 1.8+

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/backend/java.html)
{% endhint %}



入门

示例

#### 创建 `PasswordlessClient` 实例： <a href="#create-passwordlessclient-instance" id="create-passwordlessclient-instance"></a>

注册通行密钥

验证用户

定制化

示例

此 Java 实现与 Java 1.8 及更高版本兼容。[注册](../api.md#register-token)函数可能看起来像这样：

```java
import java.io.IOException;
import java.io.InputStream;
import java.io.PrintWriter;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.Scanner;

public class CreateToken {

    // 定义用于与远程服务器进行身份验证的 API 密钥
    private static final String API_SECRET = "YOUR_API_SECRET";

    public static void main(String[] args) throws IOException {
        // 从命令行参数获取别名
        String alias = args[0];

        // 使用目标 URL 创建 URL 对象
        URL url = new URL("https://v4.passwordless.dev/register/token");
        // 打开到指定 URL 的 HTTP 连接
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        // 设置请求方式为 POST
        connection.setRequestMethod("POST");
        // 为 API 密钥和内容类型设置请求标头
        connection.setRequestProperty("ApiSecret", API_SECRET);
        connection.setRequestProperty("Content-Type", "application/json");

        // 将请求的 JSON payload 构建为字符串
        String payload = "{" +
                "  \"userId\": " + getRandomInt() + "," +
                "  \"username\": \"" + alias + "\", " +
                "  \"aliases\": [\"" + alias + "\"]" +
                "}";

        // 启用连接输出以允许写入数据
        connection.setDoOutput(true);
        // 使用 PrintWriter 将 payload 写入连接的输出流
        try (PrintWriter writer = new PrintWriter(connection.getOutputStream())) {
            writer.print(payload);
        }

        // 从连接中获取响应代码和响应消息
        int responseCode = connection.getResponseCode();
        String responseMessage = connection.getResponseMessage();
        // 获取响应内容作为输入流
        InputStream responseInputStream = connection.getInputStream();

        // Read the response content using a Scanner
        Scanner scanner = new Scanner(responseInputStream);
        String responseData = scanner.nextLine();

        // 打印响应代码、消息和数据
        System.out.println("passwordless api response: " + responseCode + " " + responseMessage + " " + responseData);

        // 检查响应码是否为 200（成功）并打印接收到的令牌
        if (responseCode == 200) {
            System.out.println("received token: " + responseData);
        } else {
            // 处理或记录任何 API 错误
            // 如果需要，请在此处添加错误处理或日志记录代码
        }
    }

    // 生成随机整数值的函数
    private static int getRandomInt() {
        // 将随机浮点数（0 到 1）乘以 1e9（十亿）并返回整数值
        return (int) (1e9 * Math.random());
    }
}
```
