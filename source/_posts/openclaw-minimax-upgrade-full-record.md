---
title: "OpenClaw × MiniMax 升级全记录：从月之暗面切换，统一三大 AI 工具计费体系"
date: 2026-05-08 00:30:00
tags: [OpenClaw, MiniMax, AI工具, 飞书机器人, 龙虾, AI工作流]
categories: [AI工具, 技术折腾]
---

![OpenClaw MiniMax 升级全记录](https://raw.githubusercontent.com/MengYiXin/blog-images/main/09-openclaw-minimax-upgrade.svg)

## 背景：为什么要动 OpenClaw？

我的 AI 工作流里现在有三个主力工具：

┌────────────────┬──────────────────┬────────────────┐
│ 工具            │ 模型              │ 用途            │
├────────────────┼──────────────────┼────────────────┤
│ Hermes Agent   │ MiniMax-M2.7      │ 主力 AI 助手    │
│ Claude Code    │ MiniMax-M2.7      │ 本地代码助手    │
│ OpenClaw       │ moonshot/kimi-k2.5│ 飞书 AI 机器人  │
└────────────────┴──────────────────┴────────────────┘

OpenClaw 接的是**月之暗面 moonshot**，余额按次扣费，账单不稳定，而且 API 稳定性偶有问题。

目标很明确：**把 OpenClaw 也切换到 MiniMax**，和 Hermes、Claude Code 用同一套 API key，按包月模式计费，飞书机器人照常运行。

---

## 第一阶段：升级 OpenClaw 本体

### 升级前状态

```
OpenClaw 2026.2.3-1
moonshot / kimi-k2.5
Gateway: ws://127.0.0.1:18789
飞书：已配对
```

### 开始升级

先备份配置，然后升级：

```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup_before_upgrade_20260507
npm update -g openclaw
```

### Bug 1：npm 全局安装权限问题

报错：
```
npm error code 1
npm error command failed
```

用 `--force` 强制升级，同样报错。再试 `--ignore-scripts` 跳过安装脚本：

```bash
npm install -g openclaw --ignore-scripts
```

✅ 成功安装 558 个包。

### 升级完成确认

```bash
$ openclaw --version
OpenClaw 2026.5.6 (c97b9f7)
```

---

## 第二阶段：配置 MiniMax Provider

### 修改 openclaw.json

原来 moonshot 的 provider 配置：

```json
"providers": {
  "moonshot": {
    "baseUrl": "https://api.moonshot.cn/v1",
    "api": "openai-completions",
    "models": [...]
  }
}
```

切换为 MiniMax（baseUrl 用 `api.minimaxi.com/anthropic`，API 类型用 `anthropic-messages`）：

```json
"providers": {
  "minimax": {
    "baseUrl": "https://api.minimaxi.com/anthropic",
    "api": "anthropic-messages",
    "models": [
      {
        "id": "MiniMax-M2.7",
        "name": "MiniMax M2.7",
        "reasoning": true,
        "contextWindow": 1000000,
        "maxTokens": 16384
      }
    ]
  }
}
```

**关键点**：`api` 字段必须填 `anthropic-messages`，不是 `openai-completions`。MiniMax 是 Anthropic 兼容端点，用 `openai-completions` 会报 404 `model_not_found`。

### Bug 2：模型 API 类型错误

启动后报错：

```
GatewayClientRequestError: FailoverError: HTTP 404: model_not_found
```

直接 curl 测试 MiniMax API 是通的：

```bash
curl -X POST "https://api.minimaxi.com/anthropic/v1/messages" \
  -H "Authorization: Bearer $ANTHROPIC_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"MiniMax-M2.7","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'
```

✅ 返回正常。说明是 OpenClaw 内部路由问题。

查 `openclaw config schema` 发现 API 类型枚举里有 `anthropic-messages`，切换后问题消失。

---

## 第三阶段：飞书重新配对

OpenClaw 升级后，原有飞书配对失效。飞书发消息收到：

```
您的飞书用户ID：ou_2f259a635cd68f6ccce0de41e8fa921a
配对码：S8HKK9NV
请机器人所有者使用以下命令进行批准：
openclaw pairing approve feishu S8HKK9NV
```

执行批准：

```bash
openclaw pairing approve feishu S8HKK9NV
```

✅ 飞书重新连接成功。

### Bug 3：Gateway 进程锁定

升级后重启 Gateway，端口被旧进程（PID 6708）占用：

```
Gateway failed to start: gateway already running (pid 6708); lock timeout after 5000ms
```

杀进程后重新启动：

```bash
powershell -Command "Stop-Process -Id 6708 -Force"
openclaw gateway --port 18789
```

---

## 最终配置

```json
{
  "meta": {
    "lastTouchedVersion": "2026.5.6",
    "lastTouchedAt": "2026-05-07T15:45:00.000Z"
  },
  "providers": {
    "minimax": {
      "baseUrl": "https://api.minimaxi.com/anthropic",
      "api": "anthropic-messages",
      "models": [
        {
          "id": "MiniMax-M2.7",
          "name": "MiniMax M2.7",
          "reasoning": true,
          "contextWindow": 1000000,
          "maxTokens": 16384
        }
      ]
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "minimax/MiniMax-M2.7"
      }
    }
  }
}
```

---

## 升级后状态

┌────────────────┬──────────────────────────────────────┐
│ 项目            │ 状态                                  │
├────────────────┼──────────────────────────────────────┤
│ OpenClaw 版本   │ 2026.5.6 ✅                           │
│ 默认模型        │ MiniMax-M2.7 ✅                       │
│ API 类型        │ anthropic-messages ✅                 │
│ 飞书连接        │ 正常 ✅                               │
│ Gateway         │ ws://127.0.0.1:18789 ✅              │
└────────────────┴──────────────────────────────────────┘

三套 AI 工具现在全部统一：

┌────────────────┬──────────────────┬────────────────┐
│ 工具            │ 模型              │ API 来源        │
├────────────────┼──────────────────┼────────────────┤
│ Hermes Agent   │ MiniMax-M2.7      │ MiniMax        │
│ Claude Code    │ MiniMax-M2.7      │ MiniMax        │
│ OpenClaw       │ MiniMax-M2.7      │ MiniMax        │
└────────────────┴──────────────────┴────────────────┘

---

## 升级过程中踩的坑总结

┌────┬──────────────────────────┬──────────────────────────────┐
│ #  │ 问题                     │ 解决                         │
├────┼──────────────────────────┼──────────────────────────────┤
│ 1  │ npm 权限报错             │ --ignore-scripts 跳过安装脚本 │
│ 2  │ 404 model_not_found      │ API 类型从 openai-completions│
│    │                          │ 改为 anthropic-messages     │
│ 3  │ 飞书配对失效             │ pairing approve feishu        │
│ 4  │ Gateway 端口锁定         │ Stop-Process 强制杀旧进程    │
└────┴──────────────────────────┴──────────────────────────────┘

---

## 关于计费

OpenClaw 本身不管理计费，它只是 API 调用层。MiniMax 的计费方式（按量还是包月）在 MiniMax 账户层面设置。OpenClaw 配置里 `cost: 0` 表示本地不做费用追踪，实际账单由 MiniMax 那边决定。

我选择 MiniMax 的核心原因：**和 Hermes、Claude Code 用同一套 API key，三端一致，计费可预期，方便管理**。

---

*升级时间：2026-05-07*
