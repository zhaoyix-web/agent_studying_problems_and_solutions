# 踩坑记录：Qwen3.7-Max OpenAI兼容接口SSL握手超时（httpcore.ConnectTimeout）
## 1. 问题场景
### 学习背景
黑马Agent课程实践，使用阿里云百炼Qwen3.7-Max深度思考模型，复制平台官方OpenAI兼容Python示例代码到PyCharm运行。
### 运行环境
- Python 3.13
- openai、httpx、httpcore 依赖库
- Windows系统，后台开启Clash Verge（全局代理模式、日本境外节点、系统代理开启）
### 复现操作
1. 登录阿里云百炼控制台，进入模型广场选择 `Qwen3.7-Max`；
2. 切换至「OpenAI兼容」Python代码示例，复制完整代码；
3. 填入自己的API Key，在PyCharm中执行代码；
4. 程序长时间阻塞后抛出SSL握手超时堆栈报错，网页可正常打开dashscope.aliyuncs.com域名，仅代码网络请求失败。

## 2. 原始报错信息
### 原始出错代码
```python
from openai import OpenAI
import os

client = OpenAI(
    api_key= "sk-xxx", # 个人百炼API Key
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1",
)

messages = [{"role": "user", "content": "你是谁"}]
completion = client.chat.completions.create(
    model="qwen3.7-max",
    messages=messages,
    extra_body={"enable_thinking": True},
    stream=True
)
is_answering = False
print("\n" + "=" * 20 + "思考过程" + "=" * 20)
for chunk in completion:
    if not chunk.choices:
        continue
    delta = chunk.choices[0].delta
    if hasattr(delta, "reasoning_content") and delta.reasoning_content is not None:
        if not is_answering:
            print(delta.reasoning_content, end="", flush=True)
    if hasattr(delta, "content") and delta.content:
        if not is_answering:
            print("\n" + "=" * 20 + "完整回复" + "=" * 20)
            is_answering = True
        print(delta.content, end="", flush=True)

httpcore.ConnectTimeout: _ssl.c:1018: The handshake operation timed out
完整堆栈：底层 httpx 发起 HTTPS 请求，SSL 握手阶段连接超时，无法连通阿里云百炼国内接口地址。

## 3.根因分析
Clash Verge 开启全局代理 + 系统代理，系统全局流量全部转发至日本境外节点；
dashscope.aliyuncs.com 是阿里云国内服务器域名，境外代理转发 HTTPS 握手极易丢包、超时；
openai Python 库会自动读取系统 HTTP_PROXY/HTTPS_PROXY 环境变量，不会自动跳过国内域名，强制走代理发起请求；
网页浏览器部分会自动分流国内地址，但 Python 程序无分流逻辑，直接触发连接超时。

## 4.修复方案
打开 Clash Verge 面板；
代理模式从「全局」切换为「直连」，或直接关闭「系统代理」开关；
重新运行代码，请求不再走境外代理，接口正常连通。
