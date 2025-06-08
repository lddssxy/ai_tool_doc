---
icon: message-bot
---

# 通义千问Qwen开发简介

### 通义千问（Qwen）完整使用文档

#### 1. 概述

通义千问（Qwen）是由阿里巴巴达摩院与阿里云共同研发的大规模通用多模态大语言模型系列，具备优秀的语言理解、生成与多模态处理能力，广泛应用于对话、文本生成、摘要、翻译、代码生成、图像分析、知识问答等场景。

#### 2. 模型版本

* **Qwen2.5**（开源）。提供 Base 与 Instruct 两种风格，主要规格：7B、14B 参数。支持本地部署与离线推理。
* **Qwen3 满血版**（阿里云商业服务）。混合推理架构，支持 119 种语言与方言，多模态输入（文本、图像、音频）处理，最大上下文长度可达 32K Token。
* **Qwen-Visual**。专门针对图像理解与描述优化，支持图文融合的复杂任务。

#### 3. 功能与能力

* **自然语言理解与生成**：问答、对话、写作、改写、润色、摘要。
* **机器翻译**：支持中英、英中及多语种互译。
* **代码编写与调试**：生成多种语言示例，提供错误检查建议。
* **知识问答**：结合检索增强，对专题领域拥有较高准确度。
* **多模态处理**：图像理解、图文生成、跨模态检索。

#### 4. 接入方式

**4.1 在线体验**

* **阿里云 Model Studio（百炼）·通义千问体验中心**：
  1. 登录阿里云账户，访问 [https://ai.aliyun.com/modelstudio](https://ai.aliyun.com/modelstudio)
  2. 在“模型体验”中选择“通义千问”系列，即可在线调试与调用。
* **Qwen AI Chat 在线体验**：
  * 访问 [https://chat.qwen.ai/，支持文本与图像对话交互。](https://chat.qwen.ai/%EF%BC%8C%E6%94%AF%E6%8C%81%E6%96%87%E6%9C%AC%E4%B8%8E%E5%9B%BE%E5%83%8F%E5%AF%B9%E8%AF%9D%E4%BA%A4%E4%BA%92%E3%80%82)

**4.2 API 调用**

*   **基础请求地址**：

    ```
    POST https://model.erda.cloud.alipay.com/api/v1/qwen/predict
    ```
* **通用请求参数**：
  * `model`: "qwen3" 或 "qwen2.5"
  * `messages`: 对话历史数组，格式同 ChatGPT API。
  * `temperature`: 浮点，0.0–1.0。
  * `max_tokens`: 返回长度上限。
  * `stream`: 是否开启流式返回。
*   **示例：Python SDK**

    ```python
    from aliyun_qwen_sdk import QwenClient

    client = QwenClient(api_key="YOUR_KEY", api_secret="YOUR_SECRET")
    response = client.chat(
        model="qwen3",
        messages=[{"role":"user","content":"请解释深度学习是什么？"}],
        temperature=0.2,
        max_tokens=512
    )
    print(response["choices"][0]["message"]["content"])
    ```

**4.3 本地部署**

*   **开源镜像获取**：

    ```bash
    git clone https://github.com/QwenLM/Qwen
    cd Qwen
    docker pull qwen/qwen2.5-7b:latest
    docker run --gpus all -p 8000:8000 qwen/qwen2.5-7b
    ```
*   **命令行推理示例**：

    ```bash
    curl http://localhost:8000/v1/chat/completions \
         -H "Content-Type: application/json" \
         -d '{"model":"qwen2.5","messages":[{"role":"user","content":"写一首关于春天的诗。"}]}'
    ```

#### 5. 快速开始

1. **注册与鉴权**：在阿里云控制台开通 Model Studio 服务，获取 API Key 与 Secret。
2. **环境准备**：安装 `aliyun-qwen-sdk` 或直接使用 `requests`。
3. **调用测试**：发送简单对话请求，验证网络与鉴权。
4. **本地调优**：根据业务场景裁剪 Prompt，设置适当的温度与长度参数。

#### 6. 高级功能

* **多模态输入**：在 `messages` 中加入带 `image_url` 的消息项，实现图文混合对话。
* **长上下文管理**：借助检索-摘要-合并技术，分片处理超长输入。
* **链式思考（CoT）**：提示中加入 “请分步说明” 引导模型输出推理过程。
* **知识增强（RAG）**：结合阿里云检索服务，实时注入外部知识，提升回答准确性。

#### 7. 参数说明

| 参数          | 类型      | 默认    | 说明             |
| ----------- | ------- | ----- | -------------- |
| model       | string  | qwen3 | 模型名称           |
| messages    | array   | —     | 对话历史数组         |
| temperature | float   | 0.7   | 采样温度，越低结果越确定   |
| top\_p      | float   | 0.9   | nucleus 采样截断比例 |
| max\_tokens | int     | 1024  | 最大返回 Token 数   |
| stream      | boolean | false | 是否启用流式返回       |
| stop        | array   | null  | 停止生成的标识符列表     |

#### 8. 提示词设计与示例

*   **结构化输出**：

    ```
    "请输出 JSON 格式，包含 name、age、score 三个字段。"
    ```
*   **链式思考示例**：

    ```
    "请分步说明如何证明勾股定理，然后给出最终结论。"
    ```
* **Few-shot 示例**：在同一请求内给出两个示例输入输出，提示模型模仿。

#### 9. 限制与注意事项

1. **上下文长度限制**：Qwen2.5 支持 8K Token，Qwen3 最多 32K Token。
2. **响应延迟**：超大模型响应较慢，可通过 `stream` 分段接收。
3. **数据隐私**：避免输入敏感信息，阿里云对输入有日志记录与审计。
4. **内容审核**：使用时应结合安全风控与合规策略，过滤不当内容。

#### 10. 资源与链接

* 通义千问官网： [https://tongyi.aliyun.com/](https://tongyi.aliyun.com/)
* Model Studio： [https://ai.aliyun.com/modelstudio](https://ai.aliyun.com/modelstudio)
* GitHub 仓库： [https://github.com/QwenLM/Qwen](https://github.com/QwenLM/Qwen)
* ReadTheDocs 文档： [https://qwen.readthedocs.io/zh-cn/latest/](https://qwen.readthedocs.io/zh-cn/latest/)
* API 参考： [https://help.aliyun.com/zh/model-studio/use-qwen-by-calling-api](https://help.aliyun.com/zh/model-studio/use-qwen-by-calling-api)

#### 11. 常见问题（FAQ）

**Q1. 如何提升生成质量？**\
A1. 降低 `temperature`，使用 Chain-of-Thought 提示，提供示例。

**Q2. 如何在 Python 外调用？**\
A2. 使用任意支持 HTTP POST 的工具，如 `curl`、`Postman`。

**Q3. 是否支持私有化部署？**\
A3. Qwen2.5 开源版支持私有化部署，Qwen3 满血版暂不开放镜像。
