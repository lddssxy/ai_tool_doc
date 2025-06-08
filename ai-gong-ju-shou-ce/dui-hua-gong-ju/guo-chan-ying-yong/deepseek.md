# DeepSeek

### DeepSeek 使用指引

#### 1. 概述

DeepSeek 是中国初创团队深度求索（DeepSeek）推出的通用大语言模型系列，覆盖 Base 与 Chat 两种模式，参数规模从 7 亿到 670 亿不等，训练语料包含 2 万亿中英文文本，既支持对话生成，也可用于文本理解、改写、摘要、翻译、代码生成等多种场景 ([github.com](https://github.com/deepseek-ai/DeepSeek-LLM?utm_source=chatgpt.com), [huggingface.co](https://huggingface.co/deepseek-ai/deepseek-llm-67b-base?utm_source=chatgpt.com))。DeepSeek-R1 系列进一步在推理能力和复杂任务上做了强化，拥有 671B 参数、激活 37B 参数，并通过强化学习与知识蒸馏提升推理质量和效率 ([help.aliyun.com](https://help.aliyun.com/zh/model-studio/deepseek-api?utm_source=chatgpt.com))。

#### 2. 接入方式

* **Web 端**：访问 [chat.deepseek.com](https://chat.deepseek.com/) 或 [www.deepseek.com](https://www.deepseek.com/)（切换 “中文” 即可）即可免费体验 DeepSeek-V3 与 R1 模型 ([deepseek.com](https://www.deepseek.com/en?utm_source=chatgpt.com))。
* **移动应用**：在各大应用商店搜索 “DeepSeek Chat” ，可在手机端随时对话。
* **本地部署**：开源社区（如 Ollama）提供 DeepSeek 7B/67B Base 与 Chat 镜像，可通过命令行加载运行 ([ollama.com](https://ollama.com/library/deepseek-llm?utm_source=chatgpt.com))。

#### 3. 基本使用场景

**3.1 对话问答**

> **示例 1：常识问答**\
> **Prompt：** “请简要介绍量子纠缠的概念，并举例说明其在量子通信中的应用。”\
> **DeepSeek 响应：**\
> “量子纠缠是指两个或多个量子体系在某些物理属性上形成不可分割的整体，即使它们相距甚远，测量其中一个体系的状态会瞬时影响另一个体系的状态。在量子通信中，可通过纠缠对来实现安全密钥分发，例如量子密钥分发（QKD）协议中的 BB84 或 E91 协议，利用纠缠特性确保窃听者无法获取密钥而不被发现。”

**3.2 文本生成**

> **示例 2：文章创作**\
> **Prompt：** “写一篇 500 字的短文，主题为‘数字化转型下的中小企业机遇与挑战’。”\
> **DeepSeek 响应（开头片段）：**\
> “随着第四次工业革命的深入推进，数字化转型已成为中小企业实现跨越式发展的关键……（略）”

**3.3 文本改写与润色**

> **示例 3：学术润色**\
> **Prompt：** “请将以下句子改写得更加学术化：‘随着 AI 技术的发展，很多企业开始尝试数字化转型。’”\
> **DeepSeek 响应：**\
> “伴随着人工智能技术的不断演进，诸多企业已开始积极探索其数字化转型路径。”

**3.4 摘要与提炼**

> **示例 4：会议纪要摘要**\
> **Prompt：** “将以下会议纪要浓缩为三点关键结论：\n1. …\n2. …\n3. …”\
> **DeepSeek 响应：**\
> “- 明确了下半年项目排期及资源分配；\n- 强化了跨部门协作机制；\n- 建议引入自动化测试工具以提升交付效率。”

**3.5 机器翻译**

> **示例 5：中英互译**\
> **Prompt：** “将这段话翻译成英文：‘人工智能正在重塑各行各业的商业模式。’”\
> **DeepSeek 响应：**\
> “Artificial intelligence is reshaping the business models across various industries.”

**3.6 代码生成与调试**

> **示例 6：Python 函数编写**\
> **Prompt：** “请写一个 Python 函数，计算斐波那契数列第 n 项，并添加简单的错误检查。”\
> **DeepSeek 响应：**
>
> ```python
> def fibonacci(n):
>     if not isinstance(n, int) or n < 0:
>         raise ValueError("请输入非负整数")
>     a, b = 0, 1
>     for _ in range(n):
>         a, b = b, a + b
>     return a
> ```

#### 4. 提示词设计与高级技巧

* **指令式提示**：在 Prompt 前加入 “请按照…格式输出” 可显著提高结构化结果的质量。
* **链式思考（CoT）**：对于复杂推理题，在提示中加入 “请分步思考” 或 “先列出思路，再给出结论” 等，可引导模型给出详细推理过程 ([baoyu.io](https://baoyu.io/translations/understanding-reasoning-llms?utm_source=chatgpt.com))。
* **Few-shot 示例**：在同一 Prompt 中提供 2–3 个示例输入与输出，帮助模型学习回答风格与格式。

> **示例 7：Few-shot**\
> **Prompt：**\
> “示例：\n输入：‘今天天气如何？’\n输出：‘请提供所在城市以便查询准确天气。’\n\n输入：‘帮我写一首七言绝句。’\n输出：‘登高望远情依旧，……’\n\n输入：‘请给出中国古代四大发明及其影响。’\n输出：”

#### 5. 注意事项与最佳实践

1. **上下文长度**：DeepSeek-R1-V3 最大支持 16K Token，上下文过长时可分段喂入或摘取核心部分再处理 ([help.aliyun.com](https://help.aliyun.com/zh/model-studio/deepseek-api?utm_source=chatgpt.com))。
2. **费用控制**：合理设计 Prompt，减少无关冗余，降低调用成本。
3. **敏感信息处理**：切勿将用户隐私、商业机密直接输入模型，以免数据泄露。
4. **结果校对**：虽然模型性能优异，但仍可能出现事实性错误，务必进行人工复核。

***

以上即为 DeepSeek 在网页、应用及本地部署环境下的使用指引与示例集，涵盖常见场景、提示技巧及注意事项。可直接复制到您的“国产大模型”章节中，帮助读者快速上手并掌握 DeepSeek 的高效应用。
