---
icon: brain-circuit
---

# DeepSeek 开发指引

DeepSeek 提供强大的对话模型和代码模型供开发者使用，其中**DeepSeek Chat API**用于通用对话和自然语言处理任务，**DeepSeek Coder API**专注于编程相关的代码生成与理解任务。二者的接口设计与 OpenAI API 兼容，方便开发者集成现有工具或SDK。本文将分别介绍 Chat 与 Coder 两个模块的用途、接口调用方式、请求参数、返回格式，并通过示例代码展示如何将 DeepSeek API 集成到实际应用流程中。

### 接入与认证

在使用 DeepSeek API 前，需要在平台申请 API 密钥。所有 API 请求均通过 HTTPS 调用 `api.deepseek.com` 域名的 REST 接口，需在HTTP请求头中加入认证信息。例如，在请求头添加：`Authorization: Bearer YOUR_API_KEY`。DeepSeek API 采用 Bearer Token 认证方式，务必妥善保管好您的密钥，不要将其暴露在客户端代码仓库等场合。

DeepSeek API 默认不对并发请求数量做硬限制，并尽可能保证所有请求的服务质量。当服务器压力较大时，请求可能需要等待一段时间再返回结果。为防止连接超时，DeepSeek API 会在等待期间持续发送空行（非流式请求）或发送 SSE 心跳注释（流式请求）以保持连接活跃。如果单次请求持续超过 30 分钟仍未完成，服务器将主动关闭连接。开发者应当实现好超时和错误处理逻辑，确保应用的健壮性。

### DeepSeek Chat API

DeepSeek Chat API 提供对话补全能力，让模型基于提供的上下文消息生成回复。它适用于聊天机器人、智能问答等场景，能够理解多轮对话的上下文并生成连贯的回答。DeepSeek Chat API 当前由最新版的 DeepSeek-V3 模型提供支持（通过指定 `model="deepseek-chat"` 使用）。

#### 接口地址与调用方法

**接口路径：** `POST https://api.deepseek.com/chat/completions`\
**功能说明：** 模型根据输入的对话历史消息，生成下一步的对话回复内容。可选择一次性返回完整回复，或通过流式推送逐步返回内容。

调用时需在请求体中提供会话消息列表和目标模型等参数。接口与 OpenAI Chat Completions 相同，支持OpenAI提供的各类SDK直接调用。例如，使用 cURL 调用时格式如下：

```bash
curl https://api.deepseek.com/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
        "model": "deepseek-chat",
        "messages": [
          {"role": "system", "content": "You are a helpful assistant."},
          {"role": "user", "content": "Hello!"}
        ],
        "stream": false
      }'
```

上述示例指定了模型为 `deepseek-chat`，提供了一条系统提示和一条用户消息，要求模型非流式地生成回复。

#### 请求参数

请求体采用 JSON 格式，主要字段如下：

* **model** (string，必填)：使用的模型名称，例如 `"deepseek-chat"`。当前支持 `deepseek-chat`（通用对话模型）和 `deepseek-reasoner`（推理增强模型）。指定 `deepseek-chat` 即使用最新的 DeepSeek 对话模型。
* **messages** (array，必填)：对话消息列表，包含多轮对话的上下文。每个消息由 _role_ 和 _content_ 字段组成：
  * `role`: 发送方角色，可为`"system"`（系统指令）、`"user"`（用户）、`"assistant"`（助手）或`"tool"`（工具/函数）。其中系统消息通常用于提供模型行为规范，用户消息是用户输入，助手消息是模型回应，工具消息用于函数调用等扩展功能。
  * `content`: 消息的文本内容。例如系统消息可用于设定上下文或约束，如`"你是一个有帮助的助手"`；用户消息就是具体的问题或请求。
  * _可选_: `name` 字段可为消息指定名称（用于区分多个具有相同角色的发言者），一般无需设置。对于助手消息，还支持特殊参数如 `prefix`（布尔值，Beta特性）以强制模型回答以给定前缀开头，需要使用专门的 Beta 域名才可启用。常规使用中可忽略这些高级选项。
* **stream** (boolean，选填)：是否采用流式返回模式。默认`false`即一次性返回完整结果；设为`true`时，将通过 Server-Sent Events (text/event-stream) 流式地实时返回生成内容，最后以 `data: [DONE]` 标识流结束。
* **max\_tokens** (integer，选填)：回复的最大长度（以 token 计）。可以设置生成内容的最大 tokens 数，默认值为 4096，最大可设置为 8192，但需注意模型的输入和输出总上下文长度也受此上限限制。
* **temperature** (float，选填)：生成随机性温度，范围 0～2，默认 1.0。较高的温度会增加回复的随机性和创造力，较低的值则使输出更确定、更严谨。_提示：对于需要确定答案的任务可降低此值，例如编码或数学问答场景建议 temperature 设为 0.0 以提高确定性。_
* **top\_p** (float，选填)：核采样参数，范围 0～1，默认 1。模型会考虑概率前 top\_p 部分的词汇结果。一般建议 `temperature` 和 `top_p` 二选一调整，不要同时修改。
* **presence\_penalty** (float，选填)：出现惩罚系数，范围 -2.0～2.0，默认 0。正值会减少模型重复已有文本的倾向，鼓励输出新话题内容。
* **frequency\_penalty** (float，选填)：频率惩罚系数，范围 -2.0～2.0，默认 0。正值会降低模型重复相同词句的概率，从而减少啰嗦和复读的情况。
* **stop** (string 或 string列表，选填)：指定一个或多个终止序列。当模型生成这些序列时会停止输出，最多可提供 16 个停止词或字符串。如果不需要特殊终止条件，可不设置此参数。
* **tools** (array，选填)：提供可供模型调用的函数列表（Function Calling 功能）。目前仅支持 `"function"` 类型的工具，通过 JSON Schema 描述函数的 `name`、`description` 和 `parameters` 等。模型可能在回答中调用这些函数，以实现复杂操作。最多可注册 128 个函数供调用。
* **tool\_choice** (object，选填)：控制模型使用工具的策略。可选值包括：`"none"`（不用工具，直接回答）、`"auto"`（模型自主决定是否调用工具）或 `"required"`（必须调用至少一个工具）。也可以指定特定工具（函数）必须被调用。如果提供了函数列表而未设定此参数，默认模型会自动决定是否调用工具。
* **response\_format** (object，选填)：指定模型输出的格式。将其设为 `{"type": "json_object"}` 可开启 JSON 输出模式，要求模型严格输出 JSON 格式字符串。在需要结构化输出（如表格数据、键值对）的场景非常实用。但请确保通过提示引导模型输出JSON，否则模型可能因为尝试对齐格式而无意义地输出空白字符直至达到长度上限。
* **logprobs** (boolean，选填)：是否返回生成每个token的对数概率信息。默认不返回详细概率。当设为 true 时，返回结果中将包含 `logprobs` 字段列出模型输出中各 token 的log概率，以及 `top_logprobs`（若有设置）等信息。
* **top\_logprobs** (integer，选填)：当 `logprobs=true` 时可用。指定一个 ≤20 的整数 N，使模型在每个位置输出概率最高的 N 个 token 及其log概率。若某位置可能的token少于N，则实际返回的可能项会更少。

上述为常用主要参数，DeepSeek Chat API 与 OpenAI ChatGPT API 参数基本一致。一般情况下，只需提供 `model`、`messages` 这两个必要字段，以及根据需求调整 `max_tokens`、`temperature` 等主要选项即可。其它高级参数如函数调用、日志概率等可按需使用。

#### 返回结果格式

请求成功时，服务器返回 HTTP 200，响应内容为一个 JSON 对象，表示一次对话补全的结果。主要字段包括：

* **id**: 每次对话补全的唯一ID字符串。
* **object**: 返回对象类型，一般为 `"chat.completion"`（非流式）或 `"chat.completion.chunk"`（流式分块）。
* **created**: 时间戳（Unix秒）表示completion创建时间。
* **model**: 实际执行此补全请求的模型名称。例如返回 `"deepseek-chat"` 表示使用深度寻Seek的Chat模型完成。
* **choices**: 模型生成的回答选项列表（通常只返回一个选项，除非未来接口支持一次请求返回多回答）。每个选项包含以下字段：
  * **index**: 回答选项的索引，从0开始。
  *   **message**: 模型生成的回复消息对象，包含 `role`（通常为 `"assistant"`）和 `content`（回复内容）字段。例如：

      ```json
      "message": {
          "role": "assistant",
          "content": "Hello! How can I help you today?"
      }
      ```

      `content`即为模型生成的主要回答文本。如果启用了函数调用功能，当模型选择调用函数时，这里可能为空，转而在后续的函数调用结果中给出内容。
  * **finish\_reason**: 完成原因，指示模型停止生成的原因。可能值包括：`stop`（正常结束或遇到提供的停止符）、`length`（达到最大长度限制）、`content_filter`（内容触发过滤）、`tool_calls`（模型调用了工具/函数后停止），`insufficient_system_resource`（系统资源不足导致中止）等。
  * **logprobs**: 若请求参数开启了概率信息，则包含一个对象，列出生成内容每个 token 的log概率及Top N备选token概率等详细信息。默认为 null 不返回。
  * **tool\_calls**: 若模型调用了函数/tool（Function Calling），则返回该字段，包括函数调用的名称和参数等信息。未使用函数调用则为 null 或省略。每个函数调用包含：
    * `name`: 模型请求调用的函数名。
    * `arguments`: 模型提供的函数参数，JSON格式字符串。在执行实际函数前，开发者应验证参数的有效性。
*   **usage**: 统计此次请求的 token 使用情况，包括提示部分 (`prompt_tokens`)、回答部分 (`completion_tokens`)、总计 (`total_tokens`)，以及缓存命中与否所节省的 token 数等。例如：

    ```json
    "usage": {
        "prompt_tokens": 16,
        "completion_tokens": 10,
        "total_tokens": 26
    }
    ```

    其中 prompt\_tokens 统计提交的消息内容长度，completion\_tokens 是模型生成的回复长度。如果 DeepSeek 平台启用了上下文缓存，还会返回 `prompt_cache_hit_tokens` 等字段表示命中缓存的Token数量以供参考。

**流式响应**：当 `stream=true` 时，接口将以SSE流形式返回数据。此时HTTP响应的Content-Type为`text/event-stream`，数据被拆分为多条事件消息，每条包含部分回复内容的增量（delta）。开发者需要逐条读取并拼接这些内容。每个事件数据包含与非流式响应类似的结构，但content字段可能是分片的。例如第一条可能仅包含 `"content": "Hello"`，下一条 `"content": " World"` 等，需要按照顺序组合。此外，最后一条消息的 `data: [DONE]` 标志流结束。DeepSeek SSE返回中还会在每条增量中附带当前usage为null以及最终一条单独的usage统计（如果设置了 `include_usage` 选项）。流式模式方便实时逐字呈现模型回复，但实现上需注意解析SSE格式。

#### 使用示例

**非流式调用示例：** 以下展示如何使用 Python 调用 DeepSeek Chat API 获取一次对话回复：

```python
import requests

API_KEY = "你的DeepSeek API密钥"
url = "https://api.deepseek.com/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_KEY}"
}
data = {
    "model": "deepseek-chat",
    "messages": [
        {"role": "system", "content": "你是一个乐于助人的助手。"},
        {"role": "user", "content": "你好！"}
    ],
    "max_tokens": 100,
    "temperature": 1.0,
    "stream": False
}
response = requests.post(url, headers=headers, json=data)
result = response.json()
print(result["choices"][0]["message"]["content"])
```

在上述请求中，用户向助手说“你好！”，系统消息设定了助手的人格为乐于助人。模型会返回一个问候的回答。例如，返回结果可能类似：

```json
{
  "id": "930c60df-bf64-41c9-a88e-3ec75f81e00e",
  "object": "chat.completion",
  "created": 1705651092,
  "model": "deepseek-chat",
  "choices": [
    {
      "index": 0,
      "finish_reason": "stop",
      "message": {
        "role": "assistant",
        "content": "你好！很高兴见到你，需要我帮忙吗？"
      }
    }
  ],
  "usage": {
    "prompt_tokens": 16,
    "completion_tokens": 12,
    "total_tokens": 28
  }
}
```

可以看到，模型成功地生成了一个礼貌的中文问候回复，并给出了 token 用量统计。

**流式调用示例：** 如果将上述请求中的 `stream` 设置为 `true`，则需要处理逐步返回的数据。例如使用 Python 的 SSE 客户端或手动解析HTTP流。当 `stream=true` 时，响应会像下面这样逐条返回内容：

```
data: {"choices": [{"delta": {"role": "assistant"}, ... }]}

data: {"choices": [{"delta": {"content": "你好"} ... }]}

data: {"choices": [{"delta": {"content": "！很高兴见到你"} ... }]}

data: {"choices": [{"delta": {"content": "，需要我帮忙吗？"} ... }]}

data: {"choices": [{"finish_reason": "stop", ...} ...]}

data: [DONE]
```

开发者需拼接每个 `delta` 内的content段落才能得到完整回复 `"你好！很高兴见到你，需要我帮忙吗？"`，然后处理 `[DONE]` 完成收尾。DeepSeek 的 SSE 保持与 OpenAI API 的行为一致，方便应用平滑迁移。

### DeepSeek Coder API

DeepSeek Coder API 提供针对编程场景优化的模型能力。**DeepSeek-Coder** 模型专门用于代码相关任务，可以生成代码片段、进行代码调试和提供代码解释等。开发者利用 Coder 模型可以获得更符合编程语境的回答，例如更准确的代码建议、更少的语法错误，并能在更短时间内完成编码任务。DeepSeek-Coder 尤其擅长处理长代码脚本，能够在较大上下文中保持对整个代码的理解和引用。

值得注意的是，自 DeepSeek V2.5 版本起，官方已将原 Chat 模型与 Coder 模型合并为统一的模型，从而同时具备通用对话和代码能力。也就是说，无论指定 `deepseek-chat` 还是 `deepseek-coder`，实际上访问的都是同一底层模型（在V2.5及以上版本中）。这种设计保证了向后兼容，已有使用 deepseek-coder 名称的集成仍可继续工作，同时模型的代码能力也融入到了最新版本中。因此**DeepSeek Coder API**的调用方式与 Chat API 十分相似，开发者仅需在请求中将模型名称改为 `"deepseek-coder"` 即可。下面将具体介绍其接口使用方式和特点。

#### 调用方式与参数

**接口路径：** 同样使用 `POST https://api.deepseek.com/chat/completions` 进行调用，仅需将请求 JSON 中的 `"model"` 切换为 `"deepseek-coder"`。因为 DeepSeek API 采用统一的聊天补全接口来支持不同模型，这一点与 OpenAI API 根据模型名称自动路由类似。

**请求参数：** Coder 模型接受的参数种类和格式基本与 Chat 模型一致，包括 `messages` 列表及各类生成设置。一般推荐提供合适的系统消息来引导代码风格，例如可以在 system 提示中说明使用哪种编程语言、返回代码时无需额外解释等，以使模型输出符合预期。

编码场景下，尤其要注意以下事项：

* **Temperature 设置：** 在代码生成或调试等任务中，通常希望输出更确定、可靠的结果。因此建议将 `temperature` 设为较低值，如 0 或接近0，以减少随机性。这会让 DeepSeek-Coder 模型更严格地遵循已学到的编程知识，避免产生随机风格或不确定的代码。若需要模型给出创意性的实现方案，可适当提高 temperature，但一般不超过 0.5。
* **消息内容格式：** 用户消息中可以直接描述需求或附上代码片段让模型分析。例如：
  * 请求代码生成：`{"role": "user", "content": "请用Python实现快速排序算法。"}`
  * 请求代码解释：`{"role": "user", "content": "这段Python代码的作用是什么？```<代码片段>```"}`\
    DeepSeek-Coder 会根据提示返回相应的代码或解释。**提示：** 为确保模型明确任务，可以在 system 消息中指定：「你是一名资深Python开发工程师，只需返回代码，不要多余解释」等。
* **上下文长度：** Coder 模型同样支持长上下文输入。如果需要它阅读并改进一段大型代码，确保将整段代码作为消息内容发送，并注意 `max_tokens` 设置足够大以容纳输出。如果代码非常长，建议利用DeepSeek上下文缓存功能或分段提问，以避免超出单次请求的上下文限制。

**返回结果：** DeepSeek-Coder 的响应JSON结构与 Chat API 相同，也会返回一个包含choices的对象列表。区别在于，assistant角色的 content 通常会是代码段（可能用Markdown格式包裹），或是与编程相关的解释说明。在需要函数调用时，Coder 模型也可以像 Chat 模型那样使用 `tool_calls` 字段请求执行特定操作。

例如，当请求 DeepSeek-Coder 生成代码时，返回的 content 可能如下：

````json
"message": {
  "role": "assistant",
  "content": "```python\ndef quicksort(arr):\n    if len(arr) <= 1:\n        return arr\n    pivot = arr[len(arr)//2]\n    left = [x for x in arr if x < pivot]\n    middle = [x for x in arr if x == pivot]\n    right = [x for x in arr if x > pivot]\n    return quicksort(left) + middle + quicksort(right)\n```"
}
````

模型将代码放在Markdown的代码块标记\`\`\`\`\`内，并提供了一个Python实现的快速排序函数。开发者可以提取content字段中的代码文本并呈现给用户。

再比如，请求DeepSeek-Coder解释一段代码，可能返回：

```json
"message": {
  "role": "assistant",
  "content": "这段代码实现了快速排序算法。首先，选择数组中间的元素作为 pivot（基准）；然后使用列表解析将数组分为三部分：小于 pivot 的元素放在 left 列表，等于 pivot 的放在 middle 列表，大于 pivot 的放在 right 列表。最后递归地对 left 和 right 列表排序，并将结果与 middle 列表合并，形成排好序的数组。"
}
```

可以看到，Coder 模型能够给出详尽的代码功能解释（当用户要求解释时），并且行文风格贴近编程教学。

总之，DeepSeek Coder API 通过更专业的语料和微调，在代码相关任务上表现出色。开发者调用时大部分用法与 Chat API 相同，**关键在于选择合适的模型**并调整参数以满足代码场景需求。需要强调的是，由于 Chat 和 Coder 现已合并，用 `deepseek-chat` 也能处理代码任务，但使用 `deepseek-coder` 作为模型标识能确保调用的是针对代码优化的配置（兼容旧接口名，功能上等价于最新模型）。建议在代码生成场景中采用 Coder 模型名称，并遵循以上提示获得最佳效果。

#### 使用示例

下面通过一个综合例子演示如何结合 DeepSeek Chat 和 Coder API 来完成实际应用中的一次交互流程。例如，用户提问需要既有概念讲解又要代码示例的复杂请求，开发者可以分别调用 Chat 模型和 Coder 模型各司其职，然后将结果整合。

**场景:** 用户询问“**什么是快速排序和归并排序的区别？能给我一个Python实现的快速排序代码吗？**”。这个请求既包含算法概念的问答，也需要提供代码示例。我们可以采用 **先解释后编码** 的方案：利用 Chat API 生成对比讲解的回答，再用 Coder API 生成代码片段，最后将两部分结果返回给用户。具体步骤如下：

1.  **调用 Chat API 获取算法区别解释：** 构造请求时，将问题的第一部分（快速排序 vs 归并排序的区别）作为用户消息发送给 `deepseek-chat` 模型，让其给出通俗易懂的解释。可以在 system 消息中要求回答尽量简洁、聚焦要点。例如：

    ```json
    {
      "model": "deepseek-chat",
      "messages": [
        {"role": "system", "content": "你是一位算法专家，请简要回答用户问题。"},
        {"role": "user", "content": "快速排序和归并排序有什么区别？"}
      ]
    }
    ```

    Chat模型可能返回类似回答：

    > **助手**：快速排序和归并排序都是常用的排序算法，但原理有所不同。快速排序采用分治策略，通过选择“基准”将数组划分为左右两部分递归排序，平均时间复杂度为O(n log n)，且就地排序占用空间小，但最坏情况下可能退化为O(n^2)。归并排序同样使用分治思想，将数组不断二分再合并，时间复杂度稳定为O(n log n)，并且可以很好地利用外部存储进行排序，但需要额外的O(n)空间来合并结果。\
    > 这个回答概括了两种算法的区别。
2.  **调用 Coder API 获取代码示例：** 接着，我们针对用户请求的第二部分，让 Coder 模型生成Python代码。构造请求时，直接询问 `"请提供一个Python实现的快速排序函数"`，并发送给 `deepseek-coder` 模型。例如：

    ```json
    {
      "model": "deepseek-coder",
      "messages": [
        {"role": "user", "content": "请提供一个Python实现的快速排序函数。"}
      ]
    }
    ```

    Coder模型会返回一个Python代码片段，例如：

    > **助手**：\`\`\`python\
    > def quicksort(arr):\
    > if len(arr) <= 1:\
    > return arr\
    > pivot = arr\[len(arr)//2]\
    > left = \[x for x in arr if x < pivot]\
    > middle = \[x for x in arr if x == pivot]\
    > right = \[x for x in arr if x > pivot]\
    > return quicksort(left) + middle + quicksort(right)
    >
    > ```
    > ```

    该代码实现了快速排序算法。
3.  **整合回答：** 最后，将 Chat API 给出的解释文本与 Coder API 提供的代码组合起来，形成完整的回答返回给用户。例如，可以在解释结尾附上代码示例：

    ````
    快速排序和归并排序的区别在于……（此处省略具体解释内容）

    **Python 实现的快速排序示例代码：**
    ```python
    def quicksort(arr):
        if len(arr) <= 1:
            return arr
        pivot = arr[len(arr)//2]
        left = [x for x in arr if x < pivot]
        middle = [x for x in arr if x == pivot]
        right = [x for x in arr if x > pivot]
        return quicksort(left) + middle + quicksort(right)
    ````

    ```
    这样用户即可在同一答案中获得对概念的理解和可运行的代码。
    ```

**实现提示：** 上述流程可以通过程序自动化。例如使用 Python：

```python
# 假设已经获得 user_question，例如：
user_question = "快速排序和归并排序有什么区别？请给我一个Python实现的快速排序。"

# 分拆问题
part1 = "快速排序和归并排序有什么区别？"
part2 = "请提供一个Python实现的快速排序函数。"

# 1. 调用 Chat API 获取解释
chat_body = {
    "model": "deepseek-chat",
    "messages": [
        {"role": "system", "content": "你是一位算法专家，请简要回答用户问题。"},
        {"role": "user", "content": part1}
    ]
}
chat_resp = requests.post(url, headers=headers, json=chat_body).json()
explanation = chat_resp["choices"][0]["message"]["content"]

# 2. 调用 Coder API 获取代码
coder_body = {
    "model": "deepseek-coder",
    "messages": [
        {"role": "user", "content": part2}
    ]
}
coder_resp = requests.post(url, headers=headers, json=coder_body).json()
code_block = coder_resp["choices"][0]["message"]["content"]

# 3. 合并结果
final_answer = explanation.strip() + "\n\n代码示例：\n" + code_block
```

通过这样的方式，开发者可以灵活地将 DeepSeek 的通用对话能力与编程专长结合起来，根据用户请求的不同方面分别调用最合适的模型，最终提供完善的答案。实际应用中，还可以依据问题内容自动判断是调用 Chat 还是 Coder（或两个都调用），进一步提升响应精准度和专业性。

***

以上内容整理自 DeepSeek 官方文档，涵盖了 DeepSeek Chat API 与 DeepSeek Coder API 的主要功能、用法示例和集成方式，希望能帮助开发者快速上手将 DeepSeek 模型能力应用到自己的项目中。随着 DeepSeek 平台的迭代，新功能（如函数调用、上下文缓存等）也在不断丰富，开发者可参考官方文档的相关章节获取更多高级用法。掌握这些 API 接口后，您就可以构建起智能对话助手、代码生成工具等各类AI应用，为用户提供强大的自然语言和编程支持。
