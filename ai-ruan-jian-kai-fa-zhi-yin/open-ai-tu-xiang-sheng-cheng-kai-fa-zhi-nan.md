# 🌵 Open AI图像生成开发指南

官方 OpenAI 图像生成开发指南 [官方文档](https://platform.openai.com/docs/guides/images)

本指南依据 OpenAI 官方文档整理，涵盖图像生成相关 API 的功能简介、参数说明、代码示例（Python 和 Java）、内容审核要求、错误处理，以及一个完整的 Python Flask 应用示例，帮助开发者快速上手 OpenAI 图像生成功能。

### 功能概述

OpenAI 的图像 API 提供了三种主要功能：

* **图像生成 (Generations)**：根据文本提示从零创建全新图像。支持使用 DALL·E 3 或 DALL·E 2 模型来将文本描述转换为图像。
* **图像编辑 (Edits)**：对已有图像进行编辑。通过提供一张原始图像和一个掩码，模型根据新的文本提示替换图像的指定区域（仅 DALL·E 2 支持）。
* **图像变体 (Variations)**：基于已有图像生成不同变体，即风格或细节上有所变化的类似图像（仅 DALL·E 2 支持）。

此外，每次调用都会经过内容审核，以确保提示和图像符合使用政策，如有违规将返回错误。下文将详细说明各功能的用途、参数和示例代码，并提供注意事项。

### 图像生成（Generations）

**功能说明：** 图像生成端点用于根据文本提示创建全新图像。开发者提供文字描述（prompt），模型会据此合成图像结果。生成的图像可以通过两种格式获取：URL 链接或 Base64 编码的数据，这由 `response_format` 参数决定。默认返回的是图像的临时 URL，注意该 URL **有效期仅约一小时**。也可以选择直接返回 Base64 字符串以供程序使用。DALL·E 3 模型（又称 `gpt-image-1`）和 DALL·E 2 模型均可用于此端点；两者在生成质量、图像大小和并发上有所区别（见注意事项）。

**输入参数：** 图像生成 API (`POST /v1/images/generations`) 常用输入参数如下：

* `prompt`（字符串，必需）：图像生成的文本描述。例如 `"A colorful sunset over the mountains"`。提示词越详细，生成结果与期望越接近。**长度限制**：DALL·E 2 模型最多 1000 字符，DALL·E 3 则支持更长的提示（最高可达约 32000 字符）。
* `model`（字符串，可选）：使用的模型名称，例如 `"dall-e-3"` 或 `"dall-e-2"`。若不指定则默认为 `"dall-e-2"`（除非使用了仅适用于 DALL·E 3 的参数）。
* `n`（整数，可选）：生成的图像数量，范围 1～10，默认 1。DALL·E 2 一次请求最多可生成 10 张图像，而 DALL·E 3 每次请求只能生成1张图像（可通过并行请求提高吞吐量）。
* `size`（字符串，可选）：生成图像的尺寸。DALL·E 2 支持 `"256x256"`、`"512x512"` 或 `"1024x1024"`；DALL·E 3 支持 `"1024x1024"` 以及长宽比为16:9的长方形 (`"1024x1536"`或`"1536x1024"`)，并默认使用`1024x1024`。较小的方形图像通常生成最快。
* `response_format`（字符串，可选）：返回格式，`"url"`（默认）或`"b64_json"`。DALL·E 2 支持通过 URL 返回图像（URL 有效期60分钟），也支持 Base64；而使用 DALL·E 3 时建议使用 Base64 获取图像数据。
* `user`（字符串，可选）：终端用户的唯一标识符，用于跟踪请求来源以协助监控滥用。添加该参数不会影响响应，但会在OpenAI日志中记录，便于日后审核和统计。

**Python 示例代码：** 使用 OpenAI 官方 Python 库调用图像生成。下面的示例将根据提示生成一张1024×1024的猫咪图像，并打印返回的图像URL：

```python
import openai

openai.api_key = "YOUR_API_KEY"  # 设置 API 密钥

response = openai.Image.create(
    prompt="a white siamese cat",  # 文本提示
    model="dall-e-3",             # 使用 DALL·E 3 模型
    n=1,                          # 请求生成1张图像
    size="1024x1024"              # 指定图像尺寸
)
image_url = response['data'][0]['url']
print("Image URL:", image_url)
```

上述代码构建了请求并调用 `openai.Image.create(...)` 来生成图像。其中 `prompt` 提供图像描述，参数 `n` 指定返回图像数目，`size` 指定图像分辨率。调用成功后，`response['data']` 列表包含生成的图像信息（默认是 URL），代码取第一张图像的 URL 并打印。

**Java 示例代码：** OpenAI 尚未提供官方 Java SDK，但可以使用HTTP客户端（如 OkHttp）直接调用 REST 接口。以下示例演示如何发送图像生成请求并获取返回结果：

```java
OkHttpClient client = new OkHttpClient();
 
// 准备JSON请求体
String json = "{"
    + "\"prompt\": \"a white siamese cat\","
    + "\"model\": \"dall-e-3\","
    + "\"n\": 1,"
    + "\"size\": \"1024x1024\""
    + "}";
RequestBody requestBody = RequestBody.create(
    json, MediaType.parse("application/json; charset=utf-8"));
 
// 创建HTTP请求
Request request = new Request.Builder()
    .url("https://api.openai.com/v1/images/generations")
    .header("Authorization", "Bearer YOUR_API_KEY")  // 添加认证密钥
    .post(requestBody)
    .build();
 
// 发送请求并获取响应
try (Response response = client.newCall(request).execute()) {
    if (response.isSuccessful()) {
        String responseJson = response.body().string();
        System.out.println("Image generation result: " + responseJson);
        // 可将responseJson解析，提取其中的图像URL或Base64数据
    } else {
        System.err.println("Request failed with status: " + response.code());
        System.err.println("Error body: " + response.body().string());
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

上述 Java 代码使用 OkHttp 创建一个 POST 请求，将 `prompt` 等参数作为 JSON 字符串发送到图像生成端点。成功时返回的 JSON 中包含 `data` 列表，其中每一项包含生成的图像链接（`url`）或 Base64 数据，以及 `created` 时间戳等信息。

### 图像编辑（Edits）

**功能说明：** 图像编辑端点用于根据新提示对现有图像进行修改（即图像**填充/替换**），也称为“图像填充 (inpainting)”。调用时需要提供原始图像、掩码图像和文本提示。掩码用于指示要替换的区域：在掩码 PNG 中，**透明区域（alpha=0）表示允许模型修改的部分**，而不透明区域将保留原样。模型会根据提供的新 `prompt` 来生成填充内容，**提示应描述修改后的整张图像，而非仅描述被替换的部分**。该功能仅支持使用 DALL·E 2 模型进行编辑。

**输入参数：** 图像编辑 API (`POST /v1/images/edits`) 在图像生成的基础上增加了必要的文件参数：

* `image`（文件，必需）：待编辑的原始图片文件。**要求**：必须为**PNG格式**，尺寸为正方形，文件大小不超过4MB。
* `mask`（文件，可选）：PNG格式的掩码图像，与原图尺寸相同。掩码中透明部分表示允许编辑的区域；非透明部分将保留不变。如果提供掩码，则只会在透明区域进行替换；如果不提供掩码，模型会尝试根据提示在整个图像范围进行编辑。
* `prompt`（字符串，必需）：描述编辑后图像的文本提示，需完整描述期望的最终图像内容。_注意：因为编辑只在指定区域发生，但提示应包含整个图像的语境，以便模型生成合理的替换部分。_
* `model`（字符串，可选）：编辑所用模型，只能为 `"dall-e-2"`（当前只有 DALL·E 2 支持编辑）。
* `n`（整数，可选）：生成的编辑结果数量，默认为1，可最多请求10个编辑变体。
* `size`（字符串，可选）：输出图像尺寸，同图像生成接口（256、512或1024正方形像素）。
* `response_format`（字符串，可选）：返回格式，同前述（URL 或 Base64）。
* `user`（字符串，可选）：终端用户标识符（同前述作用）。

**Python 示例代码：** 使用 OpenAI Python 库进行图像编辑。下面示例将一张客厅照片中的泳池区域替换为一只火烈鸟：

```python
import openai

openai.api_key = "YOUR_API_KEY"

response = openai.Image.create_edit(
    image=open("sunlit_lounge.png", "rb"),  # 原始图像文件
    mask=open("mask.png", "rb"),            # 掩码图像文件
    prompt="A sunlit indoor lounge area with a pool containing a flamingo",
    model="dall-e-2",
    n=1,
    size="1024x1024"
)
edited_url = response['data'][0]['url']
print("Edited Image URL:", edited_url)
```

在此例中，`sunlit_lounge.png` 是原图，`mask.png` 指定了要编辑的区域（透明部分为泳池位置）。提示描述了包含火烈鸟的室内泳池场景。调用成功后会返回编辑后的新图像 URL。**注意：上传的原图和掩码都**必须为正方形 PNG，大小<4MB，且二者尺寸相同。

**Java 示例代码：** 使用 OkHttp 调用 OpenAI 图像编辑接口。由于需要上传文件，须使用表单数据 (multipart/form-data) 格式发送请求：

```java
OkHttpClient client = new OkHttpClient();

// 准备图片文件和掩码文件
File imageFile = new File("sunlit_lounge.png");
File maskFile = new File("mask.png");

// 构建请求体（multipart form）
RequestBody imageBody = RequestBody.create(imageFile, MediaType.parse("image/png"));
RequestBody maskBody = RequestBody.create(maskFile, MediaType.parse("image/png"));
RequestBody multipartBody = new MultipartBody.Builder().setType(MultipartBody.FORM)
    .addFormDataPart("image", "sunlit_lounge.png", imageBody)
    .addFormDataPart("mask", "mask.png", maskBody)
    .addFormDataPart("prompt", "A sunlit indoor lounge area with a pool containing a flamingo")
    .addFormDataPart("model", "dall-e-2")
    .addFormDataPart("n", "1")
    .addFormDataPart("size", "1024x1024")
    .build();

// 创建HTTP请求
Request request = new Request.Builder()
    .url("https://api.openai.com/v1/images/edits")
    .header("Authorization", "Bearer YOUR_API_KEY")
    .post(multipartBody)
    .build();

// 执行请求并获取响应
try (Response response = client.newCall(request).execute()) {
    if (response.isSuccessful()) {
        String resultJson = response.body().string();
        System.out.println("Edit result: " + resultJson);
    } else {
        System.err.println("Edit failed: " + response.code());
        System.err.println("Error: " + response.body().string());
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

上述代码将原始图像和掩码作为表单字段上传，并附加所需的文本参数。若请求成功，返回 JSON 中 `data` 列表包含编辑后图像（同样以 URL 或 base64形式提供）。开发者可进一步解析 `resultJson` 来获取生成的图像链接用于展示或下载。

### 图像变体（Variations）

**功能说明：** 图像变体端点用于基于一张输入图片，生成若干张在内容或风格上有所变化的类似图像。这种功能适合用于创意设计中获取不同风格变换，或为给定图像生成扩展的备选方案。变体生成不需要额外的文本提示，只需提供原始图片即可。该功能也仅支持 DALL·E 2 模型。

**输入参数：** 图像变体 API (`POST /v1/images/variations`) 所需参数较少：

* `image`（文件，必需）：要生成变体的源图像文件。**要求**：PNG 格式，正方形尺寸，大小不超过4MB（与编辑接口相同）。
* `model`（字符串，可选）：仅 `"dall-e-2"` 可用。
* `n`（整数，可选）：生成的变体数量，1～10，默认为1。
* `size`（字符串，可选）：输出图像尺寸，支持256、512、1024正方形像素。
* `response_format`（字符串，可选）：返回格式，URL（默认，链接一小时内有效）或 Base64。
* `user`（字符串，可选）：终端用户标识。

**Python 示例代码：** 使用 Python 库生成图像变体。以下示例为输入图片生成一张变体图片：

```python
import openai

openai.api_key = "YOUR_API_KEY"

response = openai.Image.create_variation(
    image=open("corgi_and_cat_paw.png", "rb"),  # 源图像文件
    model="dall-e-2",
    n=1,
    size="1024x1024"
)
variant_url = response['data'][0]['url']
print("Variant Image URL:", variant_url)
```

该请求将上传 `corgi_and_cat_paw.png` 图像，并请求模型生成一张相似风格的图片。\*\*注意：\*\*上传的源图也必须为正方形 PNG，大小小于4MB。否则API会返回错误。

**Java 示例代码：** 使用 OkHttp 调用图像变体生成，与编辑接口类似但无需掩码和提示：

```java
OkHttpClient client = new OkHttpClient();
File imageFile = new File("corgi_and_cat_paw.png");

// 构建multipart请求体（仅含源图像）
RequestBody imageBody = RequestBody.create(imageFile, MediaType.parse("image/png"));
RequestBody multipartBody = new MultipartBody.Builder().setType(MultipartBody.FORM)
    .addFormDataPart("image", "corgi_and_cat_paw.png", imageBody)
    .addFormDataPart("model", "dall-e-2")
    .addFormDataPart("n", "1")
    .addFormDataPart("size", "1024x1024")
    .build();

Request request = new Request.Builder()
    .url("https://api.openai.com/v1/images/variations")
    .header("Authorization", "Bearer YOUR_API_KEY")
    .post(multipartBody)
    .build();

try (Response response = client.newCall(request).execute()) {
    if (response.isSuccessful()) {
        String json = response.body().string();
        System.out.println("Variation result: " + json);
    } else {
        System.err.println("Request failed, status: " + response.code());
        System.err.println("Error: " + response.body().string());
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

这段代码与编辑接口的Java示例类似，但去除了掩码和prompt字段，仅上传源图像并指定所需参数。成功时返回的JSON中将包含生成的变体图像信息。

### 内容审核（Moderation）

OpenAI 对通过图像API提交的**所有提示和图像**都会自动执行内容审核，以确保符合使用政策。这意味着如果提交的文本提示或图像内容涉及违规（例如色情、仇恨、暴力等不允许的内容），API 将拒绝请求并返回错误。开发者应注意：

* **提示词限制**：避免使用违反 OpenAI 内容政策的描述，否则会收到错误响应。例如，不能请求生成暴力血腥或成人性质的图片。
* **图像内容限制**：对编辑或变体接口提供的源图像也有内容要求，违规图像（如包含不当内容）同样可能触发审核拒绝。
* **审核错误响应**：当内容被审核拦截时，API 通常返回一个错误码和信息来指明原因，例如`400 Bad Request`，消息中会提及内容政策违规。开发者应该在应用中处理此类错误，向用户提示内容不被允许。

内容审核在后台自动完成，无法通过API关闭或绕过。请参考 OpenAI 的[内容政策](https://labs.openai.com/policies/content-policy)获取完整的允许/禁止内容类别清单。总之，在构造图像生成的输入时务必遵守政策，以免请求失败。

### 错误处理

调用图像生成接口可能会遇到各种错误情况，例如参数不合法、文件格式或大小不符合要求、触发内容审核限制、以及超出速率限制等。健壮的应用应对这些错误进行处理。常见的错误类型和处理建议包括：

* **无效输入错误**：如必需参数缺失，参数值超出允许范围等，API会返回HTTP 400错误，`error`字段包含具体问题描述。检查错误信息修改请求参数。
* **文件相关错误**：上传的图像文件不符合要求（非PNG、大小超限、维度不匹配等）会返回错误。确保遵循文档中图像格式要求，如正方形PNG和大小限制。
* **内容审核错误**：如果提示或图像违规，API将返回错误以及指示可能的政策违规类别。此时应该反馈给用户修改输入。
* **速率/配额限制**：如果短时间内请求过多，可能返回429 Rate Limit错误。OpenAI对图像API有默认速率限制和配额，不同模型和账号级别有所不同。开发者应遵循文档中的指南（如每分钟请求次数限制），必要时实现重试或退避策略。

**Python 错误处理示例：** 使用 try/except 捕获 OpenAI API 调用异常：

```python
import openai

openai.api_key = "YOUR_API_KEY"
try:
    response = openai.Image.create_variation(
        image=open("image_edit_mask.png", "rb"),
        model="dall-e-2",
        n=1,
        size="1024x1024"
    )
    print(response['data'][0]['url'])
except openai.OpenAIError as e:
    # 打印HTTP状态码和错误详情
    print("Error status:", e.http_status)
    print("Error details:", e.error)
```

如上代码所示，OpenAI Python SDK 的异常 `OpenAIError` 包含 `http_status` 和 `error` 信息，便于调试。开发者可根据不同错误类型采取不同处理，例如稍后重试、提示用户修改输入等。

**Java 错误处理示例：** 在 Java 调用中，可以通过检查 HTTP 响应码和响应体来处理错误。例如，在前述 OkHttp 调用示例中，我们通过 `response.isSuccessful()` 判断请求是否成功，若否则打印了 `response.code()` 以及错误消息体。开发者也可以根据需要对特定状态码进行区别处理，例如：

```java
Response response = client.newCall(request).execute();
if (!response.isSuccessful()) {
    int code = response.code();
    String errorMsg = response.body().string();
    if (code == 429) {
        // 命中速率限制，等待后重试
    } else if (code == 400) {
        // 请求参数或内容问题
    }
    // ... 根据需要处理其他状态码
}
```

通过以上方式，确保应用不会因未处理的异常而崩溃，并能给出有意义的错误提示。

### 注意事项和最佳实践

开发者在使用图像生成接口时还需注意以下事项，以确保调用成功并获得最佳效果：

* **图像格式和大小**：输入的图像文件必须为 **PNG** 格式（DALL·E 3 还接受 JPG/WEBP，但建议使用无损的 PNG），并遵守大小限制要求。DALL·E 2 模型要求输入图像为正方形，尺寸建议256×256到1024×1024之间，文件小于4MB。DALL·E 3 模型则可接受更大的文件（可达25MB）和非正方形长宽比，但输出仍为限定尺寸。
* **生成质量设置**：针对 DALL·E 3 (`gpt-image-1`) 模型，OpenAI 提供了可选的“质量”参数用于控制生成细节和速度。例如可设置 `"quality": "high"` 获取高细节图像，或 `"medium"`, `"low"` 平衡速度（默认为 `"auto"` 自动选择）。更高质量通常意味着更慢的生成速度和更高的费用。
* **URL 时效**：API 返回的图像 URL **仅短期有效**（约1小时）。如果需要长时间保存或展示生成的图像，应及时下载并存储到自己的服务器或云存储中。
* **并发与速率**：DALL·E 2 支持单请求批量生成多张图像（最多10张），适合一次获取多样化结果；DALL·E 3 每次仅能生成一张，但可以并发调用以提高吞吐。请留意 OpenAI 针对图像API的速率限制政策，根据帐号级别调整并发数，并在收到429错误时实现退避重试。
* **费用与计价**：图像生成的费用按生成的图片数量和尺寸计算，使用更高分辨率或高质量模型会消耗更多积分或账单费用。请参阅 OpenAI 定价页面了解最新的计费标准，并监控用量避免超出预算。
* **稳定性**：图像生成结果具有一定随机性，重复相同提示可能得到不同图像。为获得满意结果，可能需要调整提示词或生成多张挑选。对于编辑功能，精细调整掩码和提示描述也有助于得到理想的修改结果。

### 完整示例：使用 Flask 构建图像生成接口

最后，我们提供一个使用 Python Flask 框架构建简单图像生成Web接口的完整示例，演示如何接收用户输入并返回生成的图像结果。

假设我们要创建一个网页服务，用户提交文本描述后由后端生成相应图像并展示。

**步骤概述：**

1. **安装依赖：** 确保已安装 `openai` Python库和 Flask。
2. **设置API密钥：** 将 OpenAI API 密钥配置为环境变量或在代码中设置。
3. **定义Flask路由：** 创建一个接收POST请求的接口读取用户的图像描述。
4. **调用图像生成API：** 在路由处理函数中，使用 `openai.Image.create` 调用图像生成，将用户提供的描述作为 `prompt`。
5. **返回结果：** 获取生成的图像URL或Base64，将其返回给前端页面显示。

下面是示例代码：

```python
from flask import Flask, request, jsonify
import openai, base64

app = Flask(__name__)
openai.api_key = "YOUR_API_KEY"  # 配置 OpenAI API 密钥

@app.route('/generate_image', methods=['POST'])
def generate_image():
    data = request.get_json()
    prompt_text = data.get('prompt')
    if not prompt_text:
        return jsonify({"error": "No prompt provided"}), 400

    try:
        # 调用 OpenAI 图像生成 API
        response = openai.Image.create(
            prompt=prompt_text,
            model="dall-e-2",    # 或 "dall-e-3"，这里选择DALL·E 2示例
            n=1,
            size="512x512",      # 生成中等尺寸图像
            response_format="b64_json"  # 要求返回Base64编码结果
        )
    except openai.OpenAIError as e:
        # 处理API错误
        return jsonify({"error": str(e), "details": e.error}), 500

    # 提取Base64图像数据并返回
    image_b64 = response['data'][0]['b64_json']
    return jsonify({"image_base64": image_b64})
```

在上述 Flask 应用中，我们定义了一个`/generate_image` POST路由。客户端发送 JSON 请求，包含用户的文本描述字段`prompt`。服务端接收后，通过 `openai.Image.create` 调用 API 生成图像。为方便传输，我们指定 `response_format="b64_json"` 直接获取 Base64 编码的图像数据，然后通过 JSON 返回。这种方式省去了托管临时URL的步骤，客户端拿到 Base64 字符串后即可直接显示图像。

**前端显示：** 前端拿到返回的 `image_base64` 字段后，可以通过将其设置为一个HTML `<img>`标签的源，例如：

```html
<img src="data:image/png;base64, {{ image_base64 }}" alt="Generated Image" />
```

这样，Base64 字符串会被浏览器解码并显示为图片。完整流程中，请确保正确处理可能的错误响应，如提示不能为空、API错误等，提高接口的健壮性。

本示例演示了一个基本的图像生成Web服务。开发者可以在此基础上扩展功能，例如：增加对`n`参数的支持一次返回多图，或提供不同尺寸/模型选项让用户选择，以及对返回的结果进行缓存等。通过本文档的说明和示例代码，您应当能够迅速开始构建基于 OpenAI 图像生成接口的应用程序，把创意转化为视觉效果。祝您开发顺利！

**参考资料：**

* [OpenAI 官方 API 文档 - 图像生成指南](https://platform.openai.com/docs/guides/images)
* [Postman 官网](https://www.postman.com/)
* [HackMD 文档](https://hackmd.io/)

###

