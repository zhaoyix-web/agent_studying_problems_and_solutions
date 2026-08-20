# Ollama + Python调用踩坑全记录
环境：Windows + Ollama + Python3.13 + VSCode虚拟环境`.venv`，模型：`qwen3:8b`

## 问题1：Python openai库调用Ollama返回502错误
**现象**：
`ollama list`、`ollama run qwen3:8b`命令行可以正常对话；但是Python使用`openai`库调用接口抛出`openai.InternalServerError Error code:502`。

**原因**：
1. `openai >=1.50`新版本客户端请求逻辑改动，和Ollama的OpenAI兼容接口存在兼容性bug，大量出现502；
2. 项目存在`openai‑agents==0.22.0`包，该包硬性要求`openai v3`版本，和新版openai库存在版本冲突。

**解决方案**
方案A（降级openai库，会带来依赖冲突）
```powershell
pip install openai==1.48.0
```
> 注意：降级后`openai‑agents`包无法使用，会报版本不兼容。

方案B（最终采用，推荐，彻底规避版本冲突）
放弃openai客户端，直接使用`requests`库调用ollama原生http接口，不受openai、httpx版本干扰。
```python
import requests

def ollama_chat(prompt:str):
    url = "http://127.0.0.1:11434/v1/chat/completions"
    payload = {
        "model":"qwen3:8b",
        "messages":[{"role":"user","content": prompt}]
    }
    resp = requests.post(url, json=payload, timeout=120)
    return resp.json()["choices"][0]["message"]["content"]

if __name__ == "__main__":
    try:
        print(ollama_chat("你好"))
    except requests.exceptions.ReadTimeout:
        print("请求超时，模型加载/推理过慢")
    except Exception as e:
        print("异常：", e)
```

## 问题2：降级openai后报`TypeError: Client.__init__() got an unexpected keyword argument 'proxies'`
**现象**：openai库初始化直接崩溃，参数`proxies`不存在。

**原因**：
`openai==1.48.0`和高版本`httpx>=0.28`不兼容，httpx新版本移除了`proxies`参数，内部初始化报错。

**解决方案**
1. 降级httpx配套版本
```powershell
pip install httpx==0.27.2 httpcore==1.0.7
```
2. 更简单：直接使用requests方案，完全避开openai+httpx版本链问题。

## 问题3：requests调用出现 ReadTimeout 读取超时
**现象**：`requests.exceptions.ReadTimeout:127.0.0.1:11434 read timed out (read timeout=30)`
端口连通，请求发送成功，但是ollama长时间不给返回结果。

**原因**
1. HTTP请求到来时才现场把大模型加载进显存，qwen3:8b加载耗时远超30秒；
2. 显存不足，模型回退CPU推理，推理速度极慢；
3. ollama后台进程异常；
4. 代理软件残留干扰网络。

**解决方案**
1. **模型预热（关键步骤）**
    终端（可以保留`.venv`虚拟环境前缀）执行
    ```powershell
    ollama run qwen3:8b
    ```
    输入一句提示词，等待完整回复输出，代表模型已经载入显存。**这个终端窗口不要关闭，关闭会释放模型**。
2. 新开另一个VSCode终端运行Python代码；
3. 调大请求超时时间，把`timeout=30`改成`timeout=120`；
4. 完全退出CC‑Switch等代理软件；
5. 排查：浏览器访问 `http://127.0.0.1:11434/v1/models`，可以看到JSON代表ollama http服务正常。
6. 任务管理器查看GPU显存占用，确认模型跑在显卡而不是CPU。

## 问题4：PowerShell里直接写curl命令报错安全警告
**现象**：windows PowerShell5.1中`curl`不是真正curl程序，是系统别名`Invoke‑WebRequest`，触发IE脚本安全拦截，请求直接失败。

**解决方案**
1. 使用`curl.exe`调用真实curl程序；
2. 或者加上参数`‑UseBasicParsing`；
3. 优先直接写Python requests脚本做接口测试，避开powershell网络坑。

## 重要避坑清单
1. ollama是系统独立程序，**和Python虚拟环境`.venv`无关**；venv只管控Python包；
2. 地址写`127.0.0.1`，不要写`localhost`，Windows存在IPv6解析异常；
3. ollama OpenAI兼容接口地址末尾必须带上`/v1`；
4. `openai‑agents 0.22.0`仅支持openai v3，不能和openai>=1.0混用；
5. ollama run交互窗口一旦关闭，显存内模型被释放，下一次http请求又要重新加载模型，容易超时；
6. 大模型显存不足会自动切CPU推理，速度会非常慢。

## 最终可以直接运行的完整代码
```python
import requests

def ollama_chat(prompt: str):
    url = "http://127.0.0.1:11434/v1/chat/completions"
    payload = {
        "model": "qwen3:8b",
        "messages": [{"role": "user", "content": prompt}]
    }
    resp = requests.post(url, json=payload, timeout=120)
    return resp.json()["choices"][0]["message"]["content"]


if __name__ == "__main__":
    res = ollama_chat("你好")
    print(res)
```

### 运行流程总结
1. VSCode终端执行`ollama run qwen3:8b`，发送消息，等待完整回复，**保持窗口打开**；
2. 新建VSCode终端，激活`.venv`虚拟环境；
3. 运行上面的python代码，获取模型输出。

