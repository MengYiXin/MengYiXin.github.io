---
title: "MiniMax VL 读图终于通了：模型名不是 VL-01，是 M3"
date: 2026-06-03 16:30:00
tags:
  - Hermes Agent
  - MiniMax
  - Vision
  - 图片识别
  - 问题解决
categories: [AI工具]
---

## 背景

5 月 9 日写过一篇 [《Hermes Agent MiniMax VL 视觉修复实录》](https://mengyx.com.cn/hermes-minimax-vl-vision-fix/)，记录了当时怎么从"图片发出去模型说看不见"一路查到 Hermes 没有接 MiniMax VL 端点，最后靠 patch 源码绕了过去。

那篇文章留下的答案是：在 `auxiliary.vision.model` 里配 `MiniMax-VL-01`。

今天（2026-06-03）发现，这条路**走不通了**。

## 问题现象

今天飞书发来一张截图，Hermes 的 `vision_analyze` 工具返回：

```
Error code: 400
{'type': 'error', 'error': {'type': 'bad_request_error', 
  'message': "invalid params, unknown model 'MiniMax-VL-01' (2013)"}}
```

图片本身就是一个错误截图——自己把自己给截进去了。

## 排查过程

### 第一步：确认是模型名问题，不是 API key 问题

用 curl 直接测 MiniMax 官方端点，换了 6 种模型名格式：

```
✗ MiniMax-VL-01  →  unknown model
✗ minimax-vl-01   →  unknown model  
✗ MiniMax/VL-01   →  unknown model
✗ vl-01           →  unknown model
✗ MiniMax-VL      →  unknown model
✗ minimax-vl01     →  unknown model
```

全部失败。说明不是格式问题，是**这个模型名本身不存在于当前 API key 的权限范围内**。

### 第二步：找能通的模型名

在 Hermes Gateway（`localhost:8642`）上逐个测试：

```
✓ minimax-vl-01   →  minimax-vl-01
✓ MiniMax/VL-01   →  MiniMax/VL-01
✓ minimax_vl_01   →  minimax_vl_01
✓ MiniMax-VL02    →  MiniMax-VL02
```

有几个能通，但实际发图还是报错 400，说明能通不代表能读图——路由过去了但后端不认识。

### 第三步：换思路——查当前 session 实际可用的多模态模型

翻了一下记忆库，发现一个之前被忽略的配置：`MiniMax-M3`。

用这个模型名做视觉分析，竟然**一次性成功了**。

## 根因

`MiniMax-VL-01` 这个名字在 MiniMax 的 API 平台上**不存在**。可能是：

- 名字改了（内部重组了模型命名体系）
- 这个 API key 没有这个模型的权限
- VL-01 是一个旧的内部名称，现在对外暴露的是 `MiniMax-M3`（原生多模态）

无论如何，当前 session 的正确配置是：

```
模型名：MiniMax-M3（不是 MiniMax-VL-01）
```

## 正确读图工作流（2026-06-03 验证通过）

**路径：飞书图 → Hermes 缓存 → vision_analyze → MiniMax-M3**

1. 用户通过飞书发送图片
2. Hermes 自动缓存到本地：`C:\Users\16080\.hermes\image_cache\img_xxx.jpg`
3. 调用 `vision_analyze(image_url="C:\\Users\\16080\\.hermes\\image_cache\\img_xxx.jpg", question="详细描述图片内容")`
4. 模型返回完整图片描述

## 手动 API 备用路径（vision_analyze 失败时用）

如果以后 `vision_analyze` 又出问题，可以手动调：

```python
import base64, urllib.request, json

with open("图片路径", "rb") as f:
    img_b64 = base64.b64encode(f.read()).decode()

payload = {
    "model": "MiniMax-M3",
    "messages": [{
        "role": "user",
        "content": [
            {"type": "text", "text": "详细描述图片的所有内容"},
            {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{img_b64}"}}
        ]
    }]
}

req = urllib.request.Request(
    "http://localhost:8642/v1/chat/completions",  # Hermes Gateway
    data=json.dumps(payload).encode(),
    headers={"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"},
    method="POST"
)
with urllib.request.urlopen(req, timeout=60) as resp:
    result = json.loads(resp.read())
    print(result["choices"][0]["message"]["content"])
```

API key 从 `~/.hermes/.env` 的 `MINIMAX_API_KEY` 读取。

## 已验证失效的模型名

以下名字在当前 API key 下**不可用**，不要填进配置：

- `MiniMax-VL-01`（连字符大写）→ 400 unknown model
- `minimax-vl-01`（全小写连字符）→ 路由通但实际调用 400
- `MiniMax-VL03` → 2013 invalid params

## 更新记录

- **2026-06-03**：`vision_analyze` 现在正常工作，模型名确认为 `MiniMax-M3`。旧配置 `MiniMax-VL-01` 废弃。
- **2026-05-09**：首次修复，思路是 patch 源码接入 VL 端点，当时的模型名配置已失效。

---

修完之后图片发过来就能看了 👀