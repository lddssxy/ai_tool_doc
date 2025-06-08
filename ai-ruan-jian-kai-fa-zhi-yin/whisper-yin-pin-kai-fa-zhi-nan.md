---
description: 开发中，有需要可以留言作者更新
---

# 🦇 Open AI音频开发指南

[官方文档](https://platform.openai.com/docs/guides/text-to-speech)\
好的，我将为您整理一份适合初学者的 OpenAI 文本转语音（Text-to-Speech）官方开发指南的中文解释指引。内容将包括官方文档中的每个部分的简要介绍、示例代码（使用 Python 和 Java），并在最后提供一个完整的开发实例。\
请稍等片刻，我整理好后会尽快呈现给您。

## OpenAI 文本转语音（TTS）官方文档指南

### 目录

* [简介](https://chatgpt.com/c/6845a49f-a148-800e-81b7-a7043798a72e#%E7%AE%80%E4%BB%8B)
* [准备工作](https://chatgpt.com/c/6845a49f-a148-800e-81b7-a7043798a72e#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)
* [基本用法：发送第一个请求](https://chatgpt.com/c/6845a49f-a148-800e-81b7-a7043798a72e#%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95%E5%8F%91%E9%80%81%E7%AC%AC%E4%B8%80%E4%B8%AA%E8%AF%B7%E6%B1%82)
* [选择模型和语音](https://chatgpt.com/c/6845a49f-a148-800e-81b7-a7043798a72e#%E9%80%89%E6%8B%A9%E6%A8%A1%E5%9E%8B%E5%92%8C%E8%AF%AD%E9%9F%B3)
* [输出格式和语速调整](https://chatgpt.com/c/6845a49f-a148-800e-81b7-a7043798a72e#%E8%BE%93%E5%87%BA%E6%A0%BC%E5%BC%8F%E5%92%8C%E8%AF%AD%E9%80%9F%E8%B0%83%E6%95%B4)
* [实时流式音频输出](https://chatgpt.com/c/6845a49f-a148-800e-81b7-a7043798a72e#%E5%AE%9E%E6%97%B6%E6%B5%81%E5%BC%8F%E9%9F%B3%E9%A2%91%E8%BE%93%E5%87%BA)
* [API 使用限制与定价](https://chatgpt.com/c/6845a49f-a148-800e-81b7-a7043798a72e#api-%E4%BD%BF%E7%94%A8%E9%99%90%E5%88%B6%E4%B8%8E%E5%AE%9A%E4%BB%B7)
* [完整开发示例：从 API Key 到语音文件](https://chatgpt.com/c/6845a49f-a148-800e-81b7-a7043798a72e#%E5%AE%8C%E6%95%B4%E5%BC%80%E5%8F%91%E7%A4%BA%E4%BE%8B%E4%BB%8E-api-key-%E5%88%B0%E8%AF%AD%E9%9F%B3%E6%96%87%E4%BB%B6)

### 简介

OpenAI 文本转语音（Text-to-Speech，简称 **TTS**）API 是一个将文字转换为自然语音的接口。开发者可以通过该 API 提供文本输入，并获取由 AI 模型生成的逼真语音输出。OpenAI 的 TTS 模型当前提供 **两种变体**：

* **TTS-1**：标准实时文本转语音模型，**优化低延迟**，适合实时应用（响应速度更快，但音质相对稍低）。
* **TTS-1-HD**：高品质文本转语音模型，**优化音质**，适合追求高保真音频的场景（音质更高，但生成耗时稍长）。

此外，TTS API 内置了**六种预设声音**（可选择不同的发音风格或音色），声音名称分别为：`alloy`、`echo`、`fable`、`onyx`、`nova`、`shimmer`。开发者可以从中选择适合应用场景的声音，使生成的语音更符合预期效果。

**主要功能和应用场景：**

* **旁白与播客**：可以将博客文章、电子书或任意书面内容转换为语音进行旁白。例如，将一篇写好的博客文章转换成有声读物，扩大受众范围。
* **多语言音频输出**：支持多种语言的文本输入，将其转为相应语音输出。语言教师或内容制作者可以用它快速生成多语言的语音课程或素材，避免人工录制多个语言音频的繁琐过程。
* **实时语音输出（流式）**：支持**流式生成**语音，实现近实时的音频输出。这对于需要即时反馈的应用（如语音助手或游戏中的角色对话）非常有用，可以逐步生成并播放音频，减少等待时间、提升交互体验。

**注意：根据 OpenAI 的使用政策，使用 TTS 功能的应用需要明确告知用户语音是由 AI 合成的**而非真人，以确保用户知情。这在将生成语音用于对外的产品或内容时尤其重要。

### 准备工作

在开始使用 OpenAI 文本转语音 API 之前，请确保以下准备工作已经完成：

* **OpenAI 账户及 API Key**：您需要拥有一个 OpenAI 帐号并开通 API 使用权限（帐号需绑定信用卡或确保有足够的额度）。登录后，在用户面板找到 **“API Keys”** 选项，创建一个新的**密钥（Secret Key）**。**请务必妥善保存**该密钥，离开页面后您将无法再次查看它。稍后调用 API 时需要使用此密钥用于身份认证。
* **开发环境**：确保本地已安装开发所需的环境。例如，建议安装 Python 3.7+（如果用 Python 调用），或具备 Java 开发环境（如果用 Java 调用）。也可以直接使用命令行的 `curl` 工具发送请求。
* **依赖库（可选）**：如果使用 Python，可安装 `requests` 库用于简化 HTTP 调用；如使用 Java，可使用标准库 `HttpURLConnection` 或第三方 HTTP 客户端。对于演示，我们将主要使用 Python 的 `requests` 和 Java 原生 HTTP 库进行说明。

准备好以上内容后，就可以开始调用 OpenAI TTS API 来将文字转换为语音了。

### 基本用法：发送第一个请求

OpenAI TTS API 提供的HTTP端点为 **`POST https://api.openai.com/v1/audio/speech`**。请求需要在HTTP头中包含认证信息和内容类型，并在请求体传入必要的参数。**最少需要提供的参数**有三个：

* `model`：使用的模型名称，填写 `"tts-1"` 或 `"tts-1-hd"`。
* `input`：要转换为语音的文本字符串。
* `voice`：选择的语音名称，例如 `"alloy"`（默认值一般为 `"alloy"`）。

下面我们通过三种方式演示**基本的 API 调用**：使用 `curl` 命令、使用 Python 的 `requests` 库、以及使用 Java 的 HTTP 请求来完成相同的任务。这些示例会将 `"Hello, world!"` 这句话转换为语音并获取音频数据。

**方法一：使用 cURL 命令行调用**

```bash
curl https://api.openai.com/v1/audio/speech \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
        "model": "tts-1",
        "voice": "alloy",
        "input": "Hello, world!"
      }' \
  --output output.mp3
```

上述命令通过 `-H` 选项添加认证头，其中将 `YOUR_API_KEY` 替换为您自己的 API 密钥。请求体（`-d` 选项部分）包含 JSON 格式的参数。`--output output.mp3` 用于将返回的音频内容直接保存为文件 _output.mp3_。执行后，如果请求成功，`output.mp3` 文件即为生成的语音音频（默认为 MP3 格式）。

**方法二：使用 Python (`requests` 库) 调用**

```python
import requests
import json

API_KEY = "你的API密钥"  # 请替换为实际的 API Key
url = "https://api.openai.com/v1/audio/speech"
headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}
data = {
    "model": "tts-1",
    "voice": "alloy",
    "input": "Hello, world!"
}

response = requests.post(url, headers=headers, json=data)
if response.status_code == 200:
    # 将返回的二进制音频内容保存为文件
    with open("output.mp3", "wb") as f:
        f.write(response.content)
    print("音频已保存为 output.mp3")
else:
    print(f"请求失败，状态码: {response.status_code}, 响应: {response.text}")
```

在上述 Python 代码中，我们构建了POST请求，将 `model`、`voice` 和 `input` 作为 JSON 发送。成功后，使用 `response.content` 获得二进制音频数据并保存为本地文件 _output.mp3_。请确保网络请求库已安装 (`pip install requests`)。

**方法三：使用 Java 原生 HTTP 调用**

```java
import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;

public class OpenAITTSExample {
    public static void main(String[] args) throws IOException {
        String apiKey = "你的API密钥";  // 替换为实际 API Key
        URL url = new URL("https://api.openai.com/v1/audio/speech");
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("POST");
        conn.setRequestProperty("Authorization", "Bearer " + apiKey);
        conn.setRequestProperty("Content-Type", "application/json");
        conn.setDoOutput(true);
        // 构造请求 JSON 字符串
        String jsonBody = "{\"model\":\"tts-1\",\"voice\":\"alloy\",\"input\":\"Hello, world!\"}";
        try(OutputStream os = conn.getOutputStream()) {
            os.write(jsonBody.getBytes(StandardCharsets.UTF_8));
        }
        // 读取响应音频数据并保存为文件
        try(InputStream in = conn.getInputStream()) {
            Files.copy(in, Paths.get("output.mp3"));
        }
        System.out.println("音频已保存为 output.mp3");
    }
}
```

上述 Java 程序使用 `HttpURLConnection` 手动创建 HTTP 请求，发送 JSON 数据并将返回的音频流保存到 _output.mp3_ 文件。运行该程序后，如无错误，当前目录下将出现生成的语音文件。

以上三种方式都完成了相同的任务：调用 OpenAI TTS API，将文本转换为语音文件。如果请求成功，您现在应该已经获得了一段 `"Hello, world!"` 的语音音频。

### 选择模型和语音

OpenAI 提供了**两个模型**（TTS-1 和 TTS-1-HD）以及**六种预设语音**供我们选择。根据不同应用需求，我们可以调整这些参数：

*   **模型选择（Model）**：

    * _TTS-1_ 模型：适用于**实时应用**，如需要较低延迟的语音对话、语音助手等。它响应速度更快，但音频质量相对于 HD 稍低。
    * _TTS-1-HD_ 模型：适用于**高音质要求**的场景，例如有声读物、广播等。它输出的语音更加清晰自然，但因为模型更复杂，生成时间略长于 TTS-1。

    _模型选择小建议_：如果您的应用对响应速度要求高（如即时交互），优先使用标准模型 TTS-1；如果对音质要求更高且可以容忍稍长的等待，考虑使用 TTS-1-HD。
*   **语音选择（Voice）**：六种语音各有不同的音色和风格：

    * `alloy` – 默认声音，**稳重温和**的音色
    * `echo` – 类似回声效果，**清亮青年**音色
    * `fable` – **讲故事**语气的音色
    * `onyx` – 男性偏低沉的**深色**音色
    * `nova` – **明亮女性**音色
    * `shimmer` – **闪烁**般略带柔和感的音色

    （以上描述为方便理解，并非官方说明词，仅用来帮助初学者区分语音风格。）开发者可以根据场景选择合适的声音，比如教育应用可能选择更温和的声音，而游戏角色可能选用个性更鲜明的声音。

使用不同模型和语音非常简单，只需在请求中调整 `model` 和 `voice` 参数值。例如，将模型换为高清模型、语音换为 `nova` 的请求代码：

```python
data = {
    "model": "tts-1-hd",           # 切换为高质量模型
    "voice": "nova",              # 切换为女性风格声音
    "input": "欢迎使用 OpenAI 文本转语音 API！"
}
response = requests.post(url, headers=headers, json=data)
# 处理响应同上...
```

上例将使用 **TTS-1-HD** 模型和 **Nova** 声音来朗读中文句子“欢迎使用 OpenAI 文本转语音 API！”。TTS 模型**支持多语言文本输入**，虽然每个预设声音主要针对英语优化，但也能以多种语言输出语音。也就是说，您可以传入中文、英文、西班牙文等文本，API 会尝试用所选声音朗读相应语言的内容。事实上，这些语音使用的底层技术与 OpenAI Whisper 模型类似，**支持多达二十多种语言**的合成。请注意，不同语言的发音可能带有声音自身的语调或口音，但整体能保证可懂度。

### 输出格式和语速调整

默认情况下，OpenAI TTS API 返回的音频格式为 **MP3**（MPEG 音频）。MP3 是一种通用格式，体积小且几乎被所有设备支持。但根据应用需求，您可能需要其他格式或特定的音频参数。官方提供了\*\*`response_format`\*\* 参数，可指定输出格式。支持的格式包括：

* **MP3**：默认格式，**通用**且体积较小，适合大多数场景和设备。
* **AAC**：高级音频编码格式，**压缩效率高**，在 iOS、Android、YouTube 等平台广泛支持。如果需要在应用中**平衡质量和文件大小**，AAC 是不错的选择。
* **FLAC**：无损音频压缩格式，可以在**不降低音质**的情况下减少文件体积。适合对音质要求极高（例如档案保存）的情况，但文件会比有损格式大。
* **Opus**：一种专为网络传输设计的**低延迟高压缩**格式，可在各种比特率下保持良好音质，适用于实时通信或流媒体音频。

要获得以上不同格式，只需在请求 JSON 中增加 `"response_format"` 字段。例如，若希望获取无损音质的 FLAC 文件，可以这样设置：

```python
data = {
    "model": "tts-1",
    "voice": "alloy",
    "input": "This is a high quality audio test.",
    "response_format": "flac"   # 指定输出FLAC格式
}
response = requests.post(url, headers=headers, json=data)
with open("output.flac", "wb") as f:
    f.write(response.content)
```

上例请求了 FLAC 格式的音频，并将结果保存为 _output.flac_。类似地，可以将 `response_format` 值改为 `"aac"`、`"opus"` 等以获取对应格式的文件。如果不指定该参数，则默认返回 MP3 格式。

**语速调节（Speed）**：TTS API 还允许通过 `speed` 参数调整语音的播放速度（语速）。默认值为 `1.0` 表示正常速度。可以使用小于1的值放慢语速，或大于1的值加快语速。例如，`0.8` 大概是偏慢的语速（慢20%），`1.5` 则是偏快的语速（快50%）。可用范围约在 **0.5 倍速**（减慢一半）到 **2.0 倍速**（加快一倍）之间。下面是一个将语速降低为80%的示例：

```python
data = {
    "model": "tts-1",
    "voice": "fable",
    "input": "Reading at a slower pace for clarity.",
    "response_format": "mp3",
    "speed": 0.8    # 将语速调慢至原来的80%
}
response = requests.post(url, headers=headers, json=data)
with open("slow_output.mp3", "wb") as f:
    f.write(response.content)
```

此请求会用 `fable` 声音以较慢的速度朗读给定文本，从而生成一个名为 _slow\_output.mp3_ 的音频文件。在不同场景下（如语音学习应用希望慢速朗读，或提高旁白紧凑度希望快速朗读），`speed` 参数都可以发挥作用。

_小贴士：_ 调整输出格式和语速可能会影响音频文件大小、质量和用户体验。无损格式音质最佳但文件较大，低速朗读清晰易懂但时长变长，高速朗读可节省时间但可能难以听清。根据具体使用场景选择合适的参数组合，达到最佳平衡。

### 实时流式音频输出

对于一些互动性强的应用，如语音助手、实时翻译或游戏对话，**降低语音输出延迟**非常关键。OpenAI TTS 提供了**流式生成**语音的能力，使得音频可以在文本还未完全转换完时就开始播放，做到边生成边输出，提升实时性。

在默认模式下，调用 `/v1/audio/speech` 接口会等待整个语音处理完成后一次性返回完整音频文件。如果输入文本较长，这意味着用户需要等待一段时间才能听到语音。而**流式输出**模式下，API 将分块(chunk)传输音频数据，您的应用可以一边接收数据块一边播放，实现近实时的语音反馈。

要使用流式功能，在编程时需要逐步读取HTTP响应流。例如，在 Python 中，如果使用 OpenAI 提供的官方库，调用 `client.audio.speech.create(..., stream=True)`（或使用类似 `response.iter_content()`）可以获取一个生成器，不断产出音频数据块。官方 OpenAI Python SDK 的示例：

```python
import openai
openai.api_key = "你的API密钥"

response = openai.Audio.speech_create(
    model="tts-1",
    voice="alloy",
    input="Hello, this is a streaming test.",
    stream=True                   # 启用流式输出
)
with open("streamed_output.mp3", "wb") as f:
    for chunk in response:
        if chunk:  # 持续写入数据块
            f.write(chunk)
```

上面的示例使用 OpenAI 的 Python 库，通过设置 `stream=True` 来开启流式输出，然后将返回的音频数据流按块写入文件。当然，实际实时播放时，可在获取到第一个数据块后立即开始播放，同时继续接收后续块，而不是先全部写入文件。

如果不使用官方SDK，您也可以直接使用 `requests` 库的流式模式（`stream=True`）处理响应。例如：

```python
response = requests.post(url, headers=headers, json=data, stream=True)
with open("streamed_output.mp3", "wb") as f:
    for chunk in response.iter_content(chunk_size=8192):
        if chunk:
            f.write(chunk)
            # 您可以在这里将chunk送入音频播放缓冲，实现边下边播
```

通过上述方法，您可以实现**边生成边播放**。请注意，流式音频主要适用于需要即时反馈的场景，而对于离线批量生成音频的场合，直接获取完整文件更简单。

### API 使用限制与定价

在使用 OpenAI 文本转语音 API 时，需要了解一些**配额限制和计费标准**：

* **请求速率限制**：默认情况下，付费账户每分钟最多可调用 **50 次** `/v1/audio/speech` 接口（50 RPM）。如果超过该速率可能会收到限流错误。对于大型应用，可考虑申请更高的限额。
* **文本长度限制**：单次请求的输入文本上限约为 **4096 个字符**（约合 4KB 文本）。这一长度大约对应 **5 分钟左右的语音**（在默认语速下）。如果需要转换更长的文本，需要将其分段多次请求，然后将多段音频拼接。
*   **计费方式**：TTS 调用的费用按输入文本的字符数计算，不同模型的费率不同：

    * **标准 TTS-1 模型**：约 $0.015 美元 / 每 1000 个字符。
    * **高清 TTS-1-HD 模型**：约 $0.030 美元 / 每 1000 个字符。

    举例来说，使用标准模型转换一段 1000 字的文本约花费 $0.015，美式单位约等于 1.5 美分；高清模型则约 3 美分/1000字。这个价格相对于人工配音是非常低廉的，**平均每分钟语音成本约为 $0.015**（标准模型下）。当然，实际费用会根据您的文本长度精确按比例计算。
* **模型选择与成本考虑**：如果您是**个人开发者或小型项目**、预算有限，建议优先使用标准 TTS-1 模型以降低成本。标准模型已经能提供相当自然的语音效果。只有在对音质要求极高并且有足够预算时，再考虑使用成本较高的 TTS-1-HD 模型。

最后，请留意 OpenAI 定期更新的[官方定价信息](https://openai.com/pricing)。上面的价格是撰写本文时的标准费率，未来可能有所调整。

### 完整开发示例：从 API Key 到语音文件

本节将以一个完整的流程示例，演示如何使用 OpenAI 文本转语音 API 将一段文本转换为音频文件并播放。我们将按步骤说明，从获取 API Key 一直到收听生成的语音。

**步骤1：获取 OpenAI API Key**\
登录 OpenAI 开发者平台（[platform.openai.com](https://platform.openai.com/)），点击左侧菜单的“**View API keys**”（API 密钥）。如果没有现有密钥，点击“**Create new secret key**”生成一个新的 API Key。复制保存该密钥，在整个使用过程中将用它来验证您的身份。切记不要将 API 密钥公开在客户端代码或版本库中。

**步骤2：准备开发环境**\
确保您的电脑上安装了必要的软件。例如，安装 Python 3 并通过 `pip install requests` 安装 `requests` 库用于HTTP请求。如果希望使用 Java，也请确保项目中可以进行 HTTP POST 请求（无需特殊库也可使用标准库）。在此示例中，我们将使用 Python 语言来完成后续步骤。

**步骤3：编写代码并发送请求**\
下面是一个 Python 脚本示例，它将把一小段中文文本转换成语音：

```python
import requests

API_KEY = "在此填写您的API密钥"
url = "https://api.openai.com/v1/audio/speech"
headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}
data = {
    "model": "tts-1",                   # 使用标准实时模型
    "voice": "alloy",                  # 选择默认的 Alloy 声音
    "input": "今天天气很好，我们来体验一下 OpenAI 的语音合成功能吧！"
}

print("正在请求 OpenAI 文本转语音 API ...")
response = requests.post(url, headers=headers, json=data)
if response.status_code == 200:
    # 将响应的音频内容写入文件
    with open("result.mp3", "wb") as f:
        f.write(response.content)
    print("音频生成成功，已保存为 result.mp3")
else:
    print(f"请求失败，状态码: {response.status_code}")
    print("错误信息：", response.text)
```

运行上述脚本时，请将 `API_KEY` 替换为您自己的密钥。脚本会将输入的中文句子发送给 OpenAI TTS API，并将返回的语音保存为 _result.mp3_。如果在终端看到“音频生成成功”的提示，说明调用成功。

**步骤4：播放生成的音频**\
现在，我们得到了语音文件 _result.mp3_。您可以通过多种方式播放它：

* **使用系统默认播放器**：直接双击或使用命令行打开 _result.mp3_ 文件。例如在 Windows 上，文件会用默认音乐播放器打开；在 macOS 上，可以用 `open result.mp3` 命令播放；在 Linux 上使用 `xdg-open result.mp3` 或类似命令。
* **使用编程库播放**：如果您希望在代码中自动播放音频，可以使用相应语言的音频播放库。例如，在 Python 中可以安装 `playsound` 库，然后调用 `playsound("result.mp3")` 播放；Java 中可以使用 JavaFX 或其他音频API来加载 _result.mp3_ 并播放。

在本示例中，我们假定开发者手动打开文件收听。您应该能听到合成的语音朗读了：“今天天气很好，我们来体验一下 OpenAI 的语音合成功能吧！”。恭喜，这表示您已经成功完成了从获取密钥、调用 API 到生成并播放语音的整个流程。

***

通过上述指南，我们从官方文档出发，逐步介绍了 OpenAI 文本转语音 API 的各个方面：核心概念、如何使用、可调整的参数以及实际的开发流程。希望这些内容对初学者清晰易懂，帮助您顺利将强大的 TTS 功能集成到自己的应用中。祝您在开发中玩转 OpenAI 的语音合成功能！
