---
description: Plugin
---

# 第三方插件（Plugin）

第三方公司提供的插件，供用户使用ChatGPT插件来调用第三方公司的接口。&#x20;

## 使用方法

1. **要使用代码解释器，需要先在Setting->Beta features中激活Plugin选项**

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption><p>ChatGPT Settings截图 </p></figcaption></figure>

2. **选择GPT-4模型插件功能**

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption><p>选择插件功能</p></figcaption></figure>

3. **点击No plugins enabled->Plugin store来打开第三方插件商店。**

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption><p>打开插件商城</p></figcaption></figure>

4. **在插件商城中选择插件，点击Install进行安装。点击Developer info可以了解插件详情。**

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption><p>安装插件</p></figcaption></figure>

5. 关闭Plugin store页面，在plugin选项中选择需要在本次对话中激活的插件，点击勾选旅行订票网站Expedia。目前单个对话中可以激活最多三个第三方插件。

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption><p>选择本次对话中需要激活的插件</p></figcaption></figure>

6. **输入与插件功能相关的提示**

如图中，Expedia是一个旅行订票网站，因此在提示中我们询问了航班信息。ChatGPT会首先对用户提示进行信息提取，并且与已激活的插件的功能描述进行匹配，如果匹配成功，则会按照插件要求的输入请求格式调用插件接口。插件API接收到chatGPT请求之后会进行处理，并且返回一段json消息给ChatGPT，其中将包含用户查询的信息以及如何给用户显示回复的一些格式信息。

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption><p>输入提示</p></figcaption></figure>

7. **点击Used Expedia我们可以查看ChatGPT与当前使用插件之间的请求和返回消息**

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption><p>查看与插件之间的请求和返回消息</p></figcaption></figure>

