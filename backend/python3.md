# Python 3+

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/backend/python3.html)
{% endhint %}

Python 3.8+ 实现只需几行代码即可完成。[注册](../api.md#register-token)函数可能看起来像这样：

```python
import requests  # 导入 requests 库，用于执行 HTTP 请求
import json      # 导入 json 库，用于处理 JSON 数据
import random    # 导入 random # 导入 random 库，用于生成随机数

# 定义用于与远程服务器进行身份验证的 API 密钥
API_SECRET = "YOUR_API_SECRET"

# 生成随机整数值的函数
def get_random_int():
  # 将随机浮点数（0 到 1）乘以 1e9（10 亿）并返回整数值
  return int(1e9 * random.random())

# 使用给定的别名创建令牌的函数
def create_token(alias):
  # 为应用程序接口请求准备 payload，包括用户 ID、用户名和别名
  payload = {
    "userId": get_random_int(),
    "username": alias,
    "aliases": [alias]
  }

  # 向指定 URL 发送 POST 请求，以 JSON 格式发送 payload，并包含 API 机密和内容类型的头信息
  response = requests.post("https://v4.passwordless.dev/register/token", json=payload, headers={"ApiSecret": API_SECRET, "Content-Type": "application/json"})

  # 将 JSON 响应解码为 Python 字典
  response_data = response.json()

  # 打印响应状态代码、文本和数据
  print("passwordless api response", response.status_code, response.text, response_data)

  # 检查响应状态代码是否为 200（成功）并打印接收到的令牌
  if response.status_code == 200:
    print("received token: ", response_data["token"])
  else:
    # 处理或记录任何 API 错误
    # 如果需要，在此处添加错误处理或日志记录代码
    pass

  # 返回响应数据
  return response_data

# 检查脚本是否作为主模块运行
if __name__ == "__main__":
  # 使用别名 "alias" 调用 create_token 函数并存储响应
  response_data = create_token("alias")
```

## 参考 <a href="#references" id="references"></a>

* [GitHub 上使用 Flask 的 Python 3.8+ 应用程序示例](https://github.com/passwordless/sdk-collection/tree/main/Python%203.8-Flask)
