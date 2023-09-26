# PHP

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/backend/php.html)
{% endhint %}

此 PHP 实现与 PHP 5.6 及更高版本兼容。但是，需要注意的是，curl 库在旧版本的 PHP 中可能不可用。因此，如果您使用的是旧版本，则可能需要更新 PHP 安装。[注册](../api.md#register-token)函数可能看起来像这样：

```php
<?php

// 定义用于与远程服务器进行身份验证的 API 密钥
$API_SECRET = "YOUR_API_SECRET";

// 生成随机整数值的函数
function get_random_int() {
  // 将 mt_rand() 生成的随机数乘以 1e9（十亿）并返回整数值
  return intval(1e9 * mt_rand());
}

// 使用给定别名创建令牌的函数
function create_token($alias) {
  global $API_SECRET; // Make the API secret accessible inside the function

  // 准备 API 请求的 payload，包括用户 ID、用户名和别名
  $payload = array(
    "userId" => get_random_int(),
    "username" => $alias,
    "aliases" => array($alias)
  );

  // 初始化 cURL 会话
  $ch = curl_init();
  // 设置远程服务器的URL
  curl_setopt($ch, CURLOPT_URL, "https://v4.passwordless.dev/register/token");
  // 表明这是一个 POST 请求
  curl_setopt($ch, CURLOPT_POST, 1);
  // 将 JSON 编码的 payload 附加为 POST 数据
  curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($payload));
  // 设置 HTTP 标头，包括 API 密钥和内容类型
  curl_setopt($ch, CURLOPT_HTTPHEADER, array("ApiSecret: $API_SECRET", "Content-Type: application/json"));
  // 以字符串形式返回响应而不是打印它
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
  // （可选）禁用 SSL 证书验证（不建议用于生产）
  // curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);

  // 执行 cURL 请求并存储响应
  $response = curl_exec($ch);

  // 检查 cURL 错误并打印它们（如果有）
  if (curl_errno($ch)) {
      print("cURL error: " . curl_error($ch) . "\n");
  }

  // 获取 HTTP 响应代码
  $response_code = curl_getinfo($ch, CURLINFO_HTTP_CODE);

  // 关闭 cURL 会话
  curl_close($ch);

  // 打印 API 响应代码
  print("passwordless api response: " . $response_code . "\n");

  // 将 JSON 响应解码为关联数组
  $response_data = json_decode($response, true);
  print_r($response_data);

  // 检查响应码是否为 200（成功）并打印接收到的令牌
  if ($response_code == 200) {
    print("received token: " . $response_data["token"] . "\n");
  } else {
    // 处理或记录任何 API 错误
    // 根据需要在此块中记录或处理错误
  }

  // 返回响应数据
  return $response_data;
}

// 检查脚本是否仅使用一个参数执行
if ($argc == 2) {
  // 从命令行参数获取别名
  $alias = $argv[1];
  // 使用别名调用 create_token 函数并存储响应
  $response_data = create_token($alias);
} else {
  // 如果未使用正确数量的参数执行脚本，则打印使用说明
  print("Usage: create_token.php <alias>\n");
}
?>
```
