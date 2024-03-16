---
description: 介绍Mid journey的使用方法以及演示使用流程
---

# Midjourney快速入门

优缺点

_**优势**：对硬件没有特殊要求，不需要单独下载安装软件_

_**缺点**：需要订阅才能使用_

## 如何注册

### 1. Discord账户

Discord是一款共多人即时通讯的网页工具，理论上只要能够正常运行Chrome等常用浏览器就可以使用Discord，Midjourney是基于Discord来提供图片生成服务的，用户需要加入Midjourney的Discord的服务器来使用。

在使用之前，用户需要注册Discord并且登陆来使用Midjourney。关于Discord的更多信息请访问[Discord官网](https://discord.com/)了解。

请根据下面指引创建或验证Discord账户：

#### [创建账户](https://support.discord.com/hc/en-us/articles/360033931551-Getting-Started)

#### [验证账户](https://support.discord.com/hc/en-us/articles/6181726888215)

### 2. 订阅Midjourney计划

在使用之前还需要订阅一个Midjourney计划。

* 访问 midjourney.com/account
* 使用经过验证的 Discord 帐户登录
*   选择如下图所示的订阅计划

    <figure><img src="https://cdn.document360.io/3040c2b6-fead-4744-a3a9-d56d621c6c7e/Images/Documentation/MJ_SubscriptionTiers.png" alt=""><figcaption><p>Midjourney Plan</p></figcaption></figure>



### 3.在Discord中加入Midjourney服务器

在开始与Midjourney机器人交互之前，需要加入Midjourney服务器

* 打开 Discord 并在左侧边栏上找到服务器列表。
* 按服务器列表底部的 `+` 按钮。
* 在弹出窗口中，单击按钮 Join a Server&#x20;
* 粘贴或键入以下 URL：http://discord.gg/midjourney 并按 Join&#x20;

<figure><img src="https://cdn.document360.io/3040c2b6-fead-4744-a3a9-d56d621c6c7e/Images/Documentation/MJ_ServerJoin1.jpg" alt=""><figcaption></figcaption></figure>

![image showing step three of joining the Midjourney Discord Server](https://cdn.document360.io/3040c2b6-fead-4744-a3a9-d56d621c6c7e/Images/Documentation/MJ\_ServerJoin2.jpg)

### 4.转到任何#General 或 #Newbie 频道

在 Discord 上加入 Midjourney 服务器后，您会看到侧边栏中列出了几个频道。

![Image of the Sidebar for Midjourney's Discord server highlighting the newbie channels.](https://cdn.document360.io/3040c2b6-fead-4744-a3a9-d56d621c6c7e/Images/Documentation/MJ\_channels.jpg)

找到并选择任何标有 `general-#` 或 的 `newbie-#频道`。这些频道专为初学者设计，以开始使用Midjourney机器人。

## 如何使用

### 使用 /imagine 命令

与 Discord 上的Midjourney机器人互动需要通过使用命令来完成。命令用于创建映像、更改默认设置、监视用户信息以及执行其他任务。 _/imagine_ 命令可以根据用户输入的提示来生成图像。[更多关于Midjourney提示](https://docs.midjourney.com/prompts)

* 在消息字段中键入“/imagine prompt：”。还可以从键入“/”时弹出的可用斜杠命令列表中选择命令 `/imagine` 。
* 在 `prompt` 字段中键入要创建的图像的描述。
* 发送您的信息。机器人将解释文本提示并开始生成图像。

<figure><img src="https://cdn.document360.io/3040c2b6-fead-4744-a3a9-d56d621c6c7e/Images/Documentation/MJ_ImagineGif.gif" alt=""><figcaption></figcaption></figure>

### 接受服务条款

在生成任何图像之前，Midjourney机器人将提示用户接受服务条款。用户必须同意这些条款才能继续创建映像。

<figure><img src="https://cdn.document360.io/3040c2b6-fead-4744-a3a9-d56d621c6c7e/Images/Documentation/MJ_TOS.jpg" alt=""><figcaption></figcaption></figure>

### 图像生成过程

提交文本提示后，Midjourney机器人会处理请求，在一分钟内创建四个图像。此过程使用GPU，每次图像生成都会计入您的中途订阅中包含的 GPU 时间。要监控可用的 GPU 时间（剩余时间很快），请使用该 `/info` 命令。

<figure><img src="https://cdn.document360.io/3040c2b6-fead-4744-a3a9-d56d621c6c7e/Images/Documentation/MJ_JobProcessing.gif" alt=""><figcaption></figcaption></figure>

### 升格或创建变体

生成初始4张图像的网格后，图像网格下方将有两行按钮可用。

<figure><img src="https://cdn.document360.io/3040c2b6-fead-4744-a3a9-d56d621c6c7e/Images/Documentation/MJ_quickstart_VariationUI.jpg" alt=""><figcaption></figcaption></figure>

* `U1` `U2` `U3` `U4` 选择对应图像进行升格

升格是指放大图片，以最大尺寸生成对应图片。按钮 `U1` 用于生成一张放大尺寸的图片1。

* `V1` `V2` `V3` `V4` 选择对应图像创建变体

变体是指生成相似图片，按钮`V1`用于生成四张图片1的相似图片。

* `🔄` 重新运行

该 `🔄` 按钮将重新运行原始提示，生成新的图像网格

### 增强或修改图片

在挑出图像后，将提供一组扩展选项。

<figure><img src="https://cdn.document360.io/3040c2b6-fead-4744-a3a9-d56d621c6c7e/Images/Documentation/MJ_UpscaledUI.png" alt=""><figcaption></figcaption></figure>

`🪄 Vary (Strong)` `🪄 Vary (Subtle)表示为所选图像创建强烈或者微小的变化的图片，一次点击会生成包含4张图片的网格。`

`🔍 Zoom Out 2x` `🔍 Zoom Out 1.5x` `🔍 Custom Zoom会缩小当前图像，`不更改原始图像内容的情况下扩展画布的原始边界。新展开的画布将使用提示和原始图像的指导进行填充。

`⬅️` ➡️ `⬆️` `⬇️` 平移按钮允许用户沿所选方向展开图像的画布，而无需更改原始图像的内容。新展开的画布将使用提示和原始图像的指导进行填充。

红色爱心用于标记最佳图片，以便在用户的midhourney网站中找到他们。

### 保存图片

单击图像以将其打开为全尺寸，然后右键单击并选择 `Save image` 。在移动设备上，长按图像，然后点击右上角的下载图标。

所有图像都可以立即在 midjourney.com/app 上查看

### 直接向midjourney机器人发送消息

在与Midjourney服务器上的其他用户一起使用时，有时会比较难找到自己提示生成的图片，这种情况可以直接点击与Midjourney机器人进行一对一的交互。在直接消息中创建的图像仍受内容和审核规则的约束，并在 Midjourney 网站上可见。

## 编写提示

即使是只有单个词的提示也可以用 Midjourney 的默认风格生成漂亮的图像，但我们通过组合艺术媒介、历史时期、位置等概念可以创建更有趣的个性化结果。

### 学习编写图片生成提示

当前midjourney只接受英文提示语，手写Midjourney提示的[基本规则](https://docs.midjourney.com/docs/explore-prompting)。

### 提示辅助生成

#### 利用ChatGPT创建Midjourney提示

利用ChatGPT创建Midjourney提示，我们给ChatGPT的提示可以分为三部分

1. Midjourney背景描述，因为ChatGPT的训练数据中并未包含Midjourney相关内容，所以在提示中我们需要让ChatGPT知道生成提示的应用场景和具体使用方式。
2. 用中文描述我们想要生成的图片
3. 提供一些优质的示例提示

下面是一个示例ChatGPT提示：

```
Midjourney是一款生成式AI，能够像Dall-e2那样将自然语言提示转换成图像。请参考示例提示，并帮助我生成一个英文
提示来满足我的图像需求。
示例：
1. A cloud with several airplanes flying around on top, in the style of detailed fantasy art, nightcore, quiet moments captured in paint, radiant clusters, i cant believe how beautiful this is, detailed character design, dark cyan and light crimson
2. ethereal Bohemian Waxwing bird, Bombycilla garrulus :: intricate details, ornate, detailed illustration, octane render :: Johanna Rupprecht style, William Morris style :: trending on artstation –ar 9:16
3. The Moon Knight dissolving into swirling sand, volumetric dust, cinematic lighting, close up portrait –ar 1:2 —q 2
要求：
1. 一个女孩骑着扫帚在月亮下飞翔
2. 女孩旁边有一个龙猫
3. 人物和动物的表情正常，手和四肢数量正常。
```

ChatGPT输出：

基于您的要求和给定的示例，以下是一个可能的英文提示：

```
"A young girl soaring beneath the moon on a broomstick, with a Totoro companion by her side, both exhibiting natural expressions and a standard number of limbs and fingers. Rendered in a detailed fantasy style, capturing the serene ambiance of the night, exuding enchantment and mystique –ar 16:9."
这个提示描述了一个详细的幻想风格的场景，并确保了人物和动物的表情与手、肢体数量的准确性。
```

#### 如何获取示例提示

在[prompthero](https://prompthero.com/midjourney-prompts)中可以找到许多示例图片和其对应的提示如下图。

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>prompthero中图片和用于生成它的提示：<br>a colorful building, in the style of intricate cityscapes, cartoon mis-en-scene, photo-realistic landscapes, cinquecento, orientalism, 32k uhd, nostalgic illustration --ar 101:128 --s 750 --niji 5</p></figcaption></figure>

#### 如何设置图片参数

上面提示中如--ar 101:128 这样的是图片的参数，--ar是指纵横比。详细参数介绍参见：[midjourney参数](https://docs.midjourney.com/docs/parameter-list)

利用如[Midjourney prompt helper](https://prompt.noonshot.com/)这样的工具我们也可以通过点选为我们的Midhourney提示词增加相应的图片参数。

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

