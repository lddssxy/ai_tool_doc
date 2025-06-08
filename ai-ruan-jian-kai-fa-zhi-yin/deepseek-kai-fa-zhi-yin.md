---
icon: brain-circuit
---

# DeepSeek 开发指引

### 1. 简介

DeepSeek 提供与 OpenAI 完全兼容的 Chat Completion API，可无缝替换或并行使用。当前主流模型包括：

* `deepseek-chat`：对应 DeepSeek-V3-0324，用于常规对话
* `deepseek-reasoner`：对应 DeepSeek-R1-0528，内置思维链输出以提升答案准确度 ([api-docs.deepseek.com](https://api-docs.deepseek.com/zh-cn/))

***

### 2. 前提条件

* 已申请并激活 DeepSeek API Key
* 网络环境可访问 `https://api.deepseek.com`
* 本地已安装 HTTP 客户端或 OpenAI SDK

***

### 3. 认证与配置

所有请求需携带 `Authorization: Bearer <DeepSeek API Key>`，并将 base URL 指向：

```
https://api.deepseek.com
```

（亦可设置 `https://api.deepseek.com/v1`，版本号与模型版本无关） ([api-docs.deepseek.com](https://api-docs.deepseek.com/zh-cn/))

**Python (OpenAI SDK) 配置示例：**

```python
from openai import OpenAI

client = OpenAI(
    api_key="<DeepSeek API Key>",
    base_url="https://api.deepseek.com"
)
```

**Node.js (OpenAI SDK) 配置示例：**

```javascript
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: "<DeepSeek API Key>",
  baseURL: "https://api.deepseek.com"
});
```

***

### 4. 调用对话 API

#### 4.1 参数说明

| 参数                                                                                                                    | 类型     | 说明                                                  |
| --------------------------------------------------------------------------------------------------------------------- | ------ | --------------------------------------------------- |
| `model`                                                                                                               | string | 选择模型，如 `"deepseek-chat"` 或 `"deepseek-reasoner"`    |
| `messages`                                                                                                            | array  | 对话消息列表，参见 [OpenAI 格式](https://platform.openai.com/) |
| `stream`                                                                                                              | bool   | 是否开启流式输出（默认 `false`）                                |
| 其他诸如 `max_tokens`、`temperature`、`top_p` 等同 OpenAI API ([api-docs.deepseek.com](https://api-docs.deepseek.com/zh-cn/)) |        |                                                     |

#### 4.2 示例：非流式调用

**curl**

```bash
curl https://api.deepseek.com/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <DeepSeek API Key>" \
  -d '{
        "model": "deepseek-chat",
        "messages": [
          {"role": "system", "content": "You are a helpful assistant."},
          {"role": "user", "content": "Hello!"}
        ],
        "stream": false
      }'
```

**Python**

```python
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ],
    stream=False
)
print(response.choices[0].message.content)
```

**Node.js**

```javascript
const completion = await openai.chat.completions.create({
  model: "deepseek-chat",
  messages: [{ role: "system", content: "You are a helpful assistant."}]
});
console.log(completion.choices[0].message.content);
```

***

### 5. 流式输出

开启 `stream: true` 后，可逐步接收模型输出，适用于长流程或实时展现。

**Python 流式示例：**

```python
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[{"role": "user", "content": "Tell me a story."}],
    stream=True
)

for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

***

### 6. 推理模型 (`deepseek-reasoner`)

`deepseek-reasoner` 在最终答案前，会先生成一段思维链（`reasoning_content`），供用户展示或蒸馏使用 ([api-docs.deepseek.com](https://api-docs.deepseek.com/zh-cn/guides/reasoning_model))。

#### 6.1 非流式示例

```python
resp = client.chat.completions.create(
    model="deepseek-reasoner",
    messages=[{"role": "user", "content": "9.11 and 9.8, which is greater?"}]
)
reasoning = resp.choices[0].message.reasoning_content
answer    = resp.choices[0].message.content
print("思维链：", reasoning)
print("答案：", answer)
```

#### 6.2 流式示例

```python
resp = client.chat.completions.create(
    model="deepseek-reasoner",
    messages=[{"role": "user", "content": "Compute 15 * 12"}],
    stream=True
)

reasoning = ""
answer    = ""
for chunk in resp:
    delta = chunk.choices[0].delta
    if hasattr(delta, "reasoning_content"):
        reasoning += delta.reasoning_content
    else:
        answer += delta.content
print("思维链：", reasoning)
print("答案：", answer)
```

***

### 7. 多轮对话

DeepSeek `/chat/completions` 为“无状态”接口，需手动拼接 `messages` 列表以保留历史 ([api-docs.deepseek.com](https://api-docs.deepseek.com/zh-cn/guides/multi_round_chat))。

```python
# Round 1
msgs = [{"role":"user","content":"What's the tallest mountain?"}]
r1 = client.chat.completions.create(model="deepseek-chat", messages=msgs)
msgs.append(r1.choices[0].message)

# Round 2
msgs.append({"role":"user","content":"And the second tallest?"})
r2 = client.chat.completions.create(model="deepseek-chat", messages=msgs)
print(r2.choices[0].message.content)
```

***

### 8. 错误处理

* **401 Unauthorized**：API Key 无效或过期，请检查 `Authorization` 头。
* **400 Bad Request**：参数错误，例如在请求中包含了不支持的 `reasoning_content`。使用推理模型时，**不可**将上次返回的 `reasoning_content` 再次发送，否则会报 400 ([api-docs.deepseek.com](https://api-docs.deepseek.com/zh-cn/guides/reasoning_model))。
* **429 Rate Limit**：请求频率超限，请根据 [限速指南](https://api-docs.deepseek.com/zh-cn/quickstart#%E9%99%90%E9%80%9F)进行重试或降速。

***

### 9. 进阶功能一览

* **JSON Output**：模型可直接输出严格 JSON 格式，适合结构化场景。
* **Function Calling**：支持与后端函数调用集成，实现动态执行。
* **上下文硬盘缓存**：通过本地缓存加速大会话上下文。
* **提示库**：官方维护的优质 Prompt 集合，便于快速开发。

详见 [API 指南](https://api-docs.deepseek.com/zh-cn/guides) ([api-docs.deepseek.com](https://api-docs.deepseek.com/zh-cn/))

***

### 10. 示例汇总

| 场景   | 调用方式       | 代码示例           |
| ---- | ---------- | -------------- |
| 基础对话 | 非流式 curl   | Section 4.2 示例 |
| 基础对话 | Python SDK | Section 4.2 示例 |
| 流式对话 | Python SDK | Section 5 示例   |
| 推理对话 | Python SDK | Section 6 示例   |
| 多轮对话 | Python SDK | Section 7 示例   |

***

> **提示**：更多模型与价格信息，请查看 [模型 & 价格](https://api-docs.deepseek.com/zh-cn/quickstart#%E6%A8%A1%E5%9E%8B-%E4%BB%B7%E6%A0%BC) ([api-docs.deepseek.com](https://api-docs.deepseek.com/zh-cn/))

***

_本章内容参考并摘编自 DeepSeek 官方文档，最新信息请以官网为准。_
