# Python 2

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/backend/python2.html)
{% endhint %}

这个 Python 2.7+ 实现只需几行代码即可完成。[注册](../api.md#register-token)函数可能看起来像这样：

```python
import requests       # 导入 requests 库，用于处理 HTTP 请求
import simplejson as json # 导入 simplejson 库并将其别名命名为 json，用于处理 JSON 数据
import random        # 导入 random 库以生成随机数

# 定义用于与远程服务器进行身份验证的 API 密钥
API_SECRET = "YOUR_API_SECRET"

# 生成随机整数值的函数
def get_random_int():
  # 用随机浮点数（0 到 1）乘以 1e9（10 亿）并返回整数值
  return int(1e9 * random.random())

# 使用给定的别名创建令牌的函数
def create_token(alias):
  # 为应用程序接口请求准备 payload，包括用户 ID、用户名和别名
  payload = {
    "userId": get_random_int(),
    "username": alias,
    "aliases": [alias]
  }

  # 使用 simplejson 的转储方法将 payload 转换为 JSON 字符串
  payload_json = json.dumps(payload)

  # 向指定的 URL 发送 POST 请求，将 JSON payload 作为数据发送，并包含 API 机密和内容类型的头信息
  response = requests.post("https://v4.passwordless.dev/register/token", data=payload_json, headers={"ApiSecret": API_SECRET, "Content-Type": "application/json"})

  # 使用 simplejson 的加载方法将 JSON 响应内容加载到 Python 字典中
  response_data = json.loads(response.content)

  # 打印响应状态代码、文本和数据
  print("passwordless api response", response.status_code, response.text, response_data)

  # 检查响应状态代码是否为 200（成功），并打印接收到的令牌
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
