---
description: 开发中，有需要可以留言作者更新
cover: >-
  https://images.unsplash.com/photo-1511497584788-876760111969?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=3432&q=80
coverY: 0
---

# 🌴 GPT API开发指南

官方文档： [GPT API 官方文档](https://platform.openai.com/docs/guides/gpt)

### GPT 开发文档 使用指南



本指南旨在帮助开发者快速上手和高效利用 OpenAI GPT 系列模型的官方开发文档，内容包括 API 注册、环境配置、接口调用、参数详解、示例代码、常见问题及优化技巧等。适用于希望将 GPT 功能集成到应用或产品中的开发者和研究人员。

***

### 1. 快速入门（Quickstart）

**目标**：在几分钟内完成环境配置，并发送第一个 API 请求。

#### 步骤示例

1.  **安装 SDK**

    ```bash
    pip install openai
    ```
2. **设置 API Key**
   *   建议在终端配置环境变量：

       ```bash
       export OPENAI_API_KEY="你的_API_KEY"
       ```
   *   或在代码中：

       ```python
       import openai
       openai.api_key = "你的_API_KEY"
       ```
3.  **发送首个请求**

    ```python
    import openai

    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": "Hello, world!"}]
    )
    print(response.choices[0].message.content)
    ```

    > 运行后若能打印出模型回复，即表明接入成功。

***

### 2. 身份验证（Authentication）

**目标**：确保请求被正确授权并避免 Key 泄露。

#### 常用方式

* **环境变量**（推荐）\
  在本地或容器中配置 `OPENAI_API_KEY`，SDK 会自动读取。
*   **硬编码**（仅限测试）

    ```python
    openai.api_key = "sk-xxx"
    ```
*   **配置文件**\
    可以把 Key 写入 `~/.openai/config`：

    ```ini
    [default]
    api_key = sk-xxx
    ```

> **提示**：切勿把硬编码 Key 提交到公共仓库，可在 `.gitignore` 中排除配置文件。

***

### 3. 速率限制与用量（Rate Limits & Usage）

**目标**：了解调用频率和成本，防止因超限导致的 429 错误。

#### 关键点

* **QPS（Queries Per Second）**：\
  不同模型有不同限制，比如免费用户对话端点通常是 3–5 QPS。
* **并发限制**：\
  同时只能发起多少个请求。
* **用量监控**：\
  在控制台（[https://platform.openai.com/account/usage）可查看每日/月度消耗。](https://platform.openai.com/account/usage%EF%BC%89%E5%8F%AF%E6%9F%A5%E7%9C%8B%E6%AF%8F%E6%97%A5/%E6%9C%88%E5%BA%A6%E6%B6%88%E8%80%97%E3%80%82)

#### 示例：捕获限流错误并退避重试

```python
import time
from openai.error import RateLimitError

for _ in range(3):
    try:
        resp = openai.ChatCompletion.create(…)
        break
    except RateLimitError:
        time.sleep(2)  # 指数退避可改为更复杂算法
```

***

### 4. 模型（Models）

**目标**：选择最适合任务需求的模型，OpenAI不断在推出新的模型，下面只是示例，请到官网查看最新模型列表。

| 模型                 | 上下文窗口      | 强项        | 建议场景       |
| ------------------ | ---------- | --------- | ---------- |
| `gpt-4`            | 8K tokens  | 高复杂度推理与创作 | 复杂对话、长文生成  |
| `gpt-4-32k`        | 32K tokens | 超长上下文     | 文档摘要、批注    |
| `gpt-3.5-turbo`    | 4K tokens  | 快速、低成本    | 聊天机器人、常见问答 |
| `text-davinci-003` | 4K tokens  | 创意写作、编辑   | 文案改写、校对    |

> **示例**：如需摘要长篇文章，可选用 `gpt-4-32k` 并将全文分段传入以保留上下文。

***

### 5. 接口参考（API Reference）

#### 5.1 聊天生成（Chat Completions）

```http
POST /v1/chat/completions
Content-Type: application/json
Authorization: Bearer $OPENAI_API_KEY

{
  "model": "gpt-3.5-turbo",
  "messages": [
    {"role": "system", "content": "你是我的助手。"},
    {"role": "user",   "content": "介绍一下巴黎。"}
  ],
  "temperature": 0.7
}
```

* **messages**：`role` 可为 `system`、`user`、`assistant`
* **temperature**：控制生成随机性

#### 5.2 文本生成（Completions）

```python
resp = openai.Completion.create(
    model="text-davinci-003",
    prompt="写一个关于人工智能的开场白。",
    max_tokens=150,
    temperature=0.6
)
```

#### 5.3 其他端点

* **Edits**：自动纠错、改写
* **Embeddings**：向量化文本，用于检索、聚类
* **Images**：生成或编辑图像
* **Moderations**：审核内容安全

***

### 6. SDK 示例（SDK Examples）

官方示例通常包含以下内容：

*   **初始化**

    ```javascript
    // Node.js
    import OpenAI from "openai";
    const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
    ```
*   **调用 Chat**

    ```javascript
    const res = await openai.chat.completions.create({
      model: "gpt-3.5-turbo",
      messages: [{ role: "user", content: "你好" }]
    });
    console.log(res.choices[0].message.content);
    ```
* **其他语言**：Go、Java、.NET 示例同理，差别主要在初始化和请求格式。

***

### 7. Playground（在线调试界面）

* **地址**：[https://platform.openai.com/playground](https://platform.openai.com/playground)
* **功能**：
  * 实时修改 `temperature`、`top_p` 等参数
  * 选择不同模型测试
  * 将调试好的参数“一键导出”成 Python / Node.js / curl 代码片段
* **应用场景**：prompt 工程、快速验证思路，无需写一行代码即可试效果。

***

### 8. CLI 工具（Command-Line Interface）

#### 安装

```bash
npm install -g openai
```

#### 常用命令

*   **创建聊天**

    ```bash
    openai api chat.completions.create \
      -m gpt-3.5-turbo \
      -g "告诉我一个笑话"
    ```
*   **补全**

    ```bash
    openai api completions.create \
      -m text-davinci-003 \
      -p "Once upon a time"
    ```

CLI 方便在脚本或 CI/CD 中快速调用，无需编写额外代码。

***

### 9. 插件与高级功能

#### 9.1 插件开发

1. **注册插件**：在控制台创建插件项目，配置插件元信息。
2. **定义 OpenAPI**：编写 `openapi.json`，描述可调用的 REST 接口。
3. **部署**：将插件托管在 HTTPS 端点，供模型按需调用。

#### 9.2 函数调用（Function Calling）

```json
"functions": [
  {
    "name": "get_current_weather",
    "description": "获取当前天气",
    "parameters": {
      "type": "object",
      "properties": {
        "location": {"type": "string", "description": "城市名称"}
      },
      "required": ["location"]
    }
  }
]
```

模型可在对话中“调用”该函数，并返回 JSON 参数，开发者根据返回内容真正调用服务。

#### 9.3 微调（Fine-tuning）

```bash
openai api fine_tunes.create \
  -t <your-file-id> \
  -m davinci \
  --n_epochs 4
```

* 上传已标注数据
* 训练完成后使用新模型 ID 进行预测

***

### 10. 安全与合规（Safety & Best Practices）

* **内容审核**：请求前后都可调用 `/v1/moderations`，拦截敏感或违规内容。
* **速率限制保护**：在高并发场景下使用队列、熔断器（Circuit Breaker）。
* **隐私保护**：
  * 不要在对话中传输用户敏感信息
  * 合规存储与清理日志

***

### 11. 常见问题（FAQ） & 支持

* **401 Unauthorized**：检查 API Key 是否正确、是否过期。
* **429 Rate Limit**：减少 QPS 或增加重试退避。
* **500 Server Error**：通常几秒后自动恢复，可重试。
* **社区支持**：
  * OpenAI 论坛：[https://community.openai.com](https://community.openai.com/)
  * GitHub Issues: [https://github.com/openai/openai-quickstart](https://github.com/openai/openai-quickstart)
* **官方文档更新**：文档常更新，建议订阅更新日志或关注 GitHub Release。

***



### 12. 开发实例：Python 简易聊天机器人

下面示例展示如何用 Python 和 `openai` SDK 构建一个基于命令行的对话机器人，将用户输入发送给 GPT-4，并打印模型回复。

```python
pythonCopyEdit# 1. 安装依赖
#    pip install openai

import os
import openai

def main():
    # 2. 设置 API Key（也可通过环境变量 OPENAI_API_KEY）
    openai.api_key = os.getenv("OPENAI_API_KEY", "你的-API-KEY")

    print("=== 欢迎使用 GPT 聊天机器人（输入 exit 或 quit 结束） ===")
    chat_history = []
    while True:
        user_input = input("你：")
        if user_input.lower() in ("exit", "quit"):
            print("再见！")
            break

        # 3. 构造对话消息列表
        chat_history.append({"role": "user", "content": user_input})
        try:
            # 4. 发起 Chat Completion 请求
            response = openai.ChatCompletion.create(
                model="gpt-4",
                messages=chat_history,
                temperature=0.7,        # 0.0-1.0，越高越“发散”
                max_tokens=500,         # 最大输出长度
                top_p=1.0,              # nucleus sampling
                frequency_penalty=0.0,  # 重复惩罚
                presence_penalty=0.6    # 新话题倾向
            )
        except openai.error.OpenAIError as e:
            print(f"[调用出错] {e}")
            continue

        # 5. 解析并展示模型回复
        reply = response.choices[0].message["content"].strip()
        chat_history.append({"role": "assistant", "content": reply})
        print(f"GPT：{reply}\n")

if __name__ == "__main__":
    main()
```

**说明**

1. **对话上下文管理**：通过 `chat_history` 将用户和模型的历史消息全部传给 API，从而实现多轮对话。
2. **参数调优**：可根据需要调整 `temperature`、`max_tokens`、`top_p` 等，优化生成质量。
3. **错误处理**：示例中捕获 `OpenAIError`，生产环境可根据错误码做更细粒度的重试或降级。



#### 13. 参考链接

* [OpenAI 官方文档](https://platform.openai.com/docs)
* [API 参考 - Chat Completions](https://platform.openai.com/docs/api-reference/chat)
* [示例项目集合](https://github.com/openai)
