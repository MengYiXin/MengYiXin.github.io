---
title: "Hermes Agent MiniMax VL 视觉修复实录：图片为什么发送后看不见"
date: 2026-05-09 19:00:00
tags:
  - Hermes Agent
  - MiniMax
  - Vision
  - 图片识别
  - Bug修复
categories: [AI工具]
---

## 问题现象

在 Hermes Agent 对话里发了一张截图，模型回复："I don't see any image attached"。

当时的第一反应是：图片格式不对？文件损坏了？于是换了一张 JPEG 图片重试——还是看不到。又试了 PNG——依然不行。

确认不是图片问题之后，开始排查 Hermes 本身的 vision 处理链路。

## 排查过程

### 第一步：排除图片本身的问题

先用工具验证图片是否正常：

```bash
# 查看图片尺寸和格式
file test_vision.jpg
# 输出：JPEG, 1004x325

# 用 tesseract OCR 提取文字（验证图片内容可读）
tesseract test_vision.jpg stdout
# 输出：成功提取出表格文字
```

图片本身没问题，尺寸、格式、EXIF 数据都正常，说明问题出在 Hermes 的图片传输环节。

### 第二步：缩小范围——是端点不支持，还是格式不兼容？

查了一下 MiniMax 的 API 文档，发现了一个关键事实：

- MiniMax **普通聊天端点** `/v1/chat/completions`：**不支持** `image_url` 格式，图片会被直接丢弃
- MiniMax **VL 端点** `/v1/coding_plan/vlm`：支持图片，但 body 格式是 `{"prompt": ..., "image_url": "data:..."}`，和 OpenAI 不一样

Hermes 的 vision tool 走的是普通聊天端点（通过 OpenAI SDK），图片以 `image_url` 格式发送——所以图片实际上**根本没有被送达**。

### 第三步：直接测试 MiniMax VL 端点

用 curl 验证 VL 端点本身是否可用：

```python
import httpx

response = httpx.post(
    "https://api.minimaxi.com/v1/coding_plan/vlm",
    headers={"Authorization": f"Bearer {MINIMAX_API_KEY}"},
    json={
        "prompt": "描述这张图的内容",
        "image_url": "data:image/jpeg;base64,..."  # 省略实际base64数据
    },
    timeout=60.0
)

print(response.json())
# 返回：成功识别图片内容
```

VL 端点本身是通的。问题只在于 Hermes 没有接入这个端点。

## 为什么 OpenClaw 能读图，Hermes 不能？

有趣的是，OpenClaw（桌面版）的图片识别是正常的。翻了它的源码后发现：OpenClaw 自己写了代码专门路由到 `/v1/coding_plan/vlm`，绕过了普通聊天端点。

Hermes 没有这个分支——它的 vision 代码只有 `_try_openai()`、`_try_anthropic()`、`_try_gemini()`、`_try_ollama()`，没有 `_try_minimax()`。

**根因确认：Hermes 的 auxiliary vision 系统不支持 MiniMax 作为 vision 后端。**

## 解决方案

### 整体思路

不通过 OpenAI SDK 调用 MiniMax，用原生 httpx 直连 VL 端点，然后手动把响应翻译成 OpenAI chat completion 格式，给下游消费者使用。

### 具体改动

修改文件：`/usr/local/lib/hermes-agent/agent/auxiliary_client.py`

#### 1. 新增 `_try_minimax_vision()` 函数

在 `_try_anthropic()` 附近插入：

```python
def _try_minimax_vision(model: str):
    api_key = os.environ.get("MINIMAX_API_KEY")
    if not api_key:
        return None
    base = os.environ.get("MINIMAX_BASE_URL", "https://api.minimaxi.com").rstrip("/")
    return _MiniMaxVLClient(api_key, base, model or "MiniMax-VL-01")
```

#### 2. 新增 `_MiniMaxVLClient` 类

核心是通过 `__getattr__` 拦截 `chat` 属性，返回一个可链式调用的代理对象：

```python
class _MiniMaxVLClient:
    def __init__(self, api_key: str, base_url: str, model: str):
        self.api_key = api_key
        self.base_url = base_url.rstrip("/")
        self.model = model
        self._chat = _ChatProxy(self)

    def __getattr__(self, name: str):
        if name == "chat":
            return self._chat
        raise AttributeError(f"no attribute '{name}'")

    def chat(self):
        return self._chat


class _ChatProxy:
    def __init__(self, client: _MiniMaxVLClient):
        self._client = client

    def __call__(self):
        return self  # 支持 client.chat() 直接调用

    def completions(self):
        return _Completions(self._client)


class _Completions:
    def create(self, messages: list[dict], **kwargs):
        # 从 messages 里提取图片（兼容 OpenAI 和 Anthropic 两种格式）
        image_data = None
        prompt_text = ""

        for msg in messages:
            content = msg.get("content", "")
            if isinstance(content, list):
                for item in content:
                    if item.get("type") == "image_url":
                        image_data = item["image_url"]["url"]
                    elif item.get("type") == "image":
                        # Hermes 内部用的是 Anthropic 格式
                        src = item.get("source", {})
                        img_data = src.get("data", "")
                        media = src.get("media_type", "image/jpeg")
                        image_data = f"data:{media};base64,{img_data}"
            elif isinstance(content, str):
                prompt_text += content + "\n"

        if not image_data:
            raise ValueError("No image data found")

        # 发往 MiniMax VL 端点
        payload = {"prompt": prompt_text.strip(), "image_url": image_data, "model": self._client.model}

        resp = httpx.post(
            f"{self._client.base_url}/v1/coding_plan/vlm",
            headers={"Authorization": f"Bearer {self._client.api_key}", "Content-Type": "application/json"},
            json=payload,
            timeout=60.0,
        )
        resp.raise_for_status()
        data = resp.json()

        # 包装成 OpenAI chat completion 格式，给下游用
        return _OpenAIMockCompletion(data)
```

#### 3. `_OpenAIMockCompletion`——把 VL 响应转成 OpenAI 格式

```python
class _OpenAIMockCompletion:
    def __init__(self, vl_data: dict):
        self.vl_data = vl_data
        self.model = vl_data.get("model", "MiniMax-VL-01")
        self.id = vl_data.get("id", "vlmock-001")
        self._text = (
            vl_data.get("response")
            or vl_data.get("output", {}).get("text")
            or (vl_data.get("choices", [{}])[0].get("message", {}).get("content", ""))
        )

    @property
    def choices(self):
        return [type("obj", (object,), {"message": type("obj", (object,), {"role": "assistant", "content": self._text})()})()]

    def model_dump(self, **kwargs):
        return {"id": self.id, "model": self.model, "choices": [{"message": {"role": "assistant", "content": self._text}}]}
```

#### 4. 修改 `_resolve_strict_vision_backend`（约 2722 行）

在 provider 判断里加 minimax：

```python
elif provider in ("minimax", "minimax-cn"):
    return _try_minimax_vision(model)
```

#### 5. 修改 `_to_async_client`（约 2083 行）

防止 `_MiniMaxVLClient` 被错误地包装成 `AsyncAnthropicAuxiliaryClient`：

```python
if isinstance(sync_client, _MiniMaxVLClient):
    return sync_client
```

#### 6. 修改 `resolve_vision_provider_client`——explicit 路径

```python
if provider in ("minimax", "minimax-cn"):
    backend, model_name = _resolve_strict_vision_backend(provider)
    if backend:
        return backend(model_name)
```

#### 7. 修改 auto 路径（`auxiliary.vision.provider: auto`）

原来 auto 路径检测到主 provider 是 minimax 时，错误地返回了 `AsyncAnthropicAuxiliaryClient`（不支持图片）。替换为：

```python
if _strict_vision_backend_available(main_provider):
    backend, model_name = _resolve_strict_vision_backend(main_provider)
    if backend:
        return backend(model_name)
```

## 验证结果

```python
asyncio.run(vision_analyze_tool(
    image_url="/tmp/test_vision.jpg",
    user_prompt="描述图片内容",
    model=None
))
```

输出（耗时 11 秒）：
> 这张图表是一份关于大语言模型（LLM）在特定硬件配置下的性能测试报告。具体来说，它展示了以下关键信息：
> 1. 测试平台：AT3500 G3 (64G) 服务器，配备了 8张华为昇腾 (Ascend) 910B AI加速卡
> 2. 测试模型：Qwen2.5-72B 和 DS-70B
> 3. 测试条件：FP16精度，30路并发...
> （后续省略）

图片内容完整识别。✅

## 坑点总结

1. **图片格式兼容性**：Hermes 内部传的是 Anthropic 格式（`{"type": "image", "source": {...}}`），不是 OpenAI 格式。`_Completions.create()` 必须同时处理两种格式。

2. **base_url 拼接**：要用 `.rstrip("/")`，否则 `f"{base}/v1/..."` 会拼成 `https://api.minimaxi.com//v1/...`（双斜杠），导致 404。

3. **`_MiniMaxVLClient` 不能用 OpenAI SDK**：SDK 会往 base_url 追加 `/chat/completions`，且发送的是 OpenAI 格式，VL 端点不认。必须用原生 httpx。

4. **版本更新会被覆盖**：这是源码层面的 hack，下次 Hermes 版本更新后需要重新 patch。建议提 PR 给官方仓库。

5. **auto 路径的陷阱**：`auxiliary.vision.provider: auto` 时，代码走的是 `resolve_provider_client` 而不是 `_resolve_strict_vision_backend`，导致 minimax 被错误地路由到不支持图片的 `AsyncAnthropicAuxiliaryClient`。

## 参考资料

- MiniMax VL 端点文档（需要申请权限）
- Hermes 源码：`agent/auxiliary_client.py`
- 之前的相关记录：[《Hermes Agent 接入腾讯 IMA 知识库》](https://mengyx.com.cn/hermes-agent-ima-tencent-integration/)

---

修完之后发图片给我就能看了 👀
