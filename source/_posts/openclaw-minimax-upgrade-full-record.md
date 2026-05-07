---
title: "OpenClaw × MiniMax 升级全记录：从月之暗面切换，统一三大 AI 工具计费体系"
date: 2026-05-08 00:30:00
tags: [OpenClaw, MiniMax, AI工具, 飞书机器人, 龙虾, AI工作流]
categories: [AI工具, 技术折腾]
---

![OpenClaw MiniMax 升级全记录](https://raw.githubusercontent.com/MengYiXin/blog-images/main/09-openclaw-minimax-upgrade.svg)

## 背景：为什么要动 OpenClaw？

我的 AI 工作流里现在有三个主力工具，全部接的 MiniMax-M2.7：

┌────────────────┬──────────────────┬────────────────┐
│ 工具            │ 模型              │ 用途            │
├────────────────┼──────────────────┼────────────────┤
│ Hermes Agent   │ MiniMax-M2.7      │ 主力 AI 助手    │
│ Claude Code    │ MiniMax-M2.7      │ 本地代码助手    │
│ OpenClaw       │ moonshot/kimi-k2.5│ 飞书 AI 机器人  │
└────────────────┴──────────────────┴────────────────┘

OpenClaw 原来跑的是**月之暗面 moonshot**，余额按次扣费，账单飘来飘去不够稳定。飞书机器人现在是我的信息入口之一，不能经常抽风。

目标很明确：**把 OpenClaw 也切换到 MiniMax**，和 Hermes、Claude Code 用同一套 API key，同一个计费体系，飞书机器人照常运行。

---

## 第一阶段：升级 OpenClaw 本体

### 升级前状态

```
OpenClaw 2026.2.3-1
Gateway: ws://127.0.0.1:18789
Provider: moonshot / kimi-k2.5
飞书：已配对
```

### Step 1：备份配置

```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup_before_upgrade_20260507
```

### Step 2：npm 全局升级

```bash
npm update -g openclaw
```

### Bug 1：npm 全局安装权限报错

```
npm error code 1
npm error command failed
npm error path ... node scripts/preinstall-package-manager-warning.mjs
npm error 'node' �����ڲ����ⲿ���Ҳ���ǿ����еĳ���
```

用 `--force` 强制升级，同样文件被锁定。改策略：先**卸载再重新安装**。

```bash
npm uninstall -g openclaw
npm install -g openclaw
```

还是报错，同一个 preinstall 脚本问题。

### 解决：--ignore-scripts 跳过安装脚本

```bash
npm install -g openclaw --ignore-scripts
```

✅ 成功安装 **558 个包**。

### Step 3：验证版本

```bash
$ openclaw --version
OpenClaw 2026.5.6 (c97b9f7)
```

从 `2026.2.3-1` 升级到了 `2026.5.6`，跨了三个月的版本。

---

## 第二阶段：修改配置，切换 Provider

### 升级前 openclaw.json 核心配置

```json
"providers": {
  "moonshot": {
    "baseUrl": "https://api.moonshot.cn/v1",
    "api": "openai-completions",
    "models": [
      {
        "id": "kimi-k2.5",
        "name": "Kimi K2.5",
        "contextWindow": 256000,
        "maxTokens": 8192
      }
    ]
  }
},
"agents": {
  "defaults": {
    "model": {
      "primary": "moonshot/kimi-k2.5"
    }
  }
}
```

### Step 1：改 provider 名和 baseUrl

baseUrl 从 `api.moonshot.cn` 换成 MiniMax 的 Anthropic 兼容端点：

```json
"providers": {
  "minimax": {
    "baseUrl": "https://api.minimaxi.com/anthropic",
    ...
  }
}
```

### Step 2：换模型

把 `kimi-k2.5` 换成 `MiniMax-M2.7`，同时大幅扩展上下文窗口：

```json
"models": [
  {
    "id": "MiniMax-M2.7",
    "name": "MiniMax M2.7",
    "reasoning": true,
    "contextWindow": 1000000,
    "maxTokens": 16384
  }
]
```

### Step 3：改默认模型

```json
"agents": {
  "defaults": {
    "model": {
      "primary": "minimax/MiniMax-M2.7"
    }
  }
}
```

### Step 4：删掉 apiKeyEnv 字段

一开始我尝试加了 `apiKeyEnv: "ANTHROPIC_API_KEY"` 指向环境变量，**但 OpenClaw 不吃这个字段**，反而导致启动报错。这条路走不通，实际测试发现 OpenClaw 会自动从环境变量 `ANTHROPIC_API_KEY` 读 key，不需额外配置。

---

## 第三阶段：API 类型——关键的坑

### Bug 2：404 model_not_found

Gateway 启动后测试 MiniMax API：

```
GatewayClientRequestError: FailoverError: HTTP 404: model_not_found
```

但直接 curl 测试 MiniMax 是通的：

```bash
curl -X POST "https://api.minimaxi.com/anthropic/v1/messages" \
  -H "Authorization: Bearer $ANTHROPIC_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"MiniMax-M2.7","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'
```

✅ 返回 `hi! 🖤`

说明问题不在 API key，也不在网络，而在 OpenClaw 内部路由。

### 根因分析

原来 moonshot 用的是 `openai-completions` API 类型。MiniMax 虽是 OpenAI 兼容端点，但这个模型 ID 在 OpenClaw 用 `openai-completions` 路由就是会 404。

### 解决：切换为 anthropic-messages

查 `openclaw config schema`，API 类型枚举里有 `anthropic-messages`。MiniMax 底层是 Anthropic 兼容接口，应该走这条路。

```json
"providers": {
  "minimax": {
    "baseUrl": "https://api.minimaxi.com/anthropic",
    "api": "anthropic-messages",
    ...
  }
}
```

重启 Gateway，问题消失。

---

## 第四阶段：飞书重新配对

OpenClaw 升级后，原有飞书配对全部失效。飞书发消息过来：

```
您的飞书用户ID：ou_2f259a635cd68f6ccce0de41e8fa921a
配对码：S8HKK9NV
请机器人所有者使用以下命令进行批准：
openclaw pairing approve feishu S8HKK9NV
```

### Step 1：批准配对

```bash
openclaw pairing approve feishu S8HKK9NV
```

### Bug 3：Gateway 进程锁死

升级后重启 Gateway，端口被旧进程（PID 6708）占用：

```
Gateway failed to start: gateway already running (pid 6708); lock timeout after 5000ms
Port 18789 is already in use.
```

tasklist 找不到进程（WSL 环境限制），用 PowerShell 强杀：

```bash
powershell -Command "Stop-Process -Id 6708 -Force"
```

再重启：

```bash
openclaw gateway --port 18789
```

### 测试飞书对话

在飞书里发消息：

```
你好
```

✅ 正常回复：`你好，Yixin 🖤`

```
你是谁
```

✅ 回复：`我是影子 🖤 你的 AI 助手。有什么需要帮忙的吗？`

```
你是openclaw 几点几？
```

✅ 回复：`OpenClaw 版本是 2026.5.6 🖤`

---

## 最终配置完整一览

`~/.openclaw/openclaw.json` 核心改动：

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
          "input": ["text"],
          "output": ["text"],
          "cacheRead": 0,
          "cacheWrite": 0,
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
      },
      "models": {
        "minimax/MiniMax-M2.7": {
          "alias": "MiniMax"
        }
      }
    }
  }
}
```

---

## 升级后状态确认

┌────────────────┬──────────────────────────────────────┐
│ 项目            │ 状态                                  │
├────────────────┼──────────────────────────────────────┤
│ OpenClaw 版本   │ 2026.5.6 ✅                           │
│ 默认模型        │ MiniMax-M2.7 ✅                       │
│ API 类型        │ anthropic-messages ✅                 │
│ 飞书连接        │ 正常 ✅                               │
│ Gateway         │ ws://127.0.0.1:18789 ✅              │
│ 身份            │ 影子 🖤 AI 助手 ✅                  │
└────────────────┴──────────────────────────────────────┘

三套 AI 工具现在全部统一用 MiniMax-M2.7：

┌────────────────┬──────────────────┬────────────────┐
│ 工具            │ 模型              │ API 来源        │
├────────────────┼──────────────────┼────────────────┤
│ Hermes Agent   │ MiniMax-M2.7      │ MiniMax        │
│ Claude Code    │ MiniMax-M2.7      │ MiniMax        │
│ OpenClaw       │ MiniMax-M2.7      │ MiniMax        │
└────────────────┴──────────────────┴────────────────┘

---

## 踩坑完整记录

┌────┬──────────────────────────┬──────────────────────────────┐
│ #  │ 问题                     │ 解决                         │
├────┼──────────────────────────┼──────────────────────────────┤
│ 1  │ npm 全局安装权限报错      │ --ignore-scripts 跳过安装脚本 │
│    │ npm error code 1         │ npm uninstall → install      │
│    │ preinstall 脚本调用 node  │ --ignore-scripts             │
│    │ 但 PATH 里 node 找不到    │                              │
├────┼──────────────────────────┼──────────────────────────────┤
│ 2  │ HTTP 404 model_not_found │ API 类型从 openai-completions │
│    │ MiniMax curl 通但        │ 改为 anthropic-messages      │
│    │ OpenClaw 内部路由 404    │ MiniMax 是 Anthropic 兼容端点 │
├────┼──────────────────────────┼──────────────────────────────┤
│ 3  │ apiKeyEnv 字段不被识别   │ 删掉该字段，OpenClaw 自动读   │
│    │ 导致启动异常             │ 环境变量 ANTHROPIC_API_KEY   │
├────┼──────────────────────────┼──────────────────────────────┤
│ 4  │ 飞书配对全部失效         │ openclaw pairing approve      │
│    │ 升级后需重新配对          │ feishu S8HKK9NV              │
├────┼──────────────────────────┼──────────────────────────────┤
│ 5  │ Gateway 端口被旧进程锁定 │ powershell Stop-Process 强杀  │
│    │ PID 6708 占用 18789      │ 再重新启动 gateway            │
└────┴──────────────────────────┴──────────────────────────────┘

---

## 关于计费

OpenClaw 本身不管理计费，它只是调用层。MiniMax 的实际计费方式（按量还是包月套餐）在 MiniMax 账户层面设置。

OpenClaw 配置里 `cost: 0` 表示本地不做费用追踪，实际账单由 MiniMax 平台决定。

统一到 MiniMax 的核心价值：**三端同一套 API key，账单可预期，方便管理**，不用在三个地方分别充值。

---

## 附：2026.5.6 相比 2026.2.3-1 的主要变化

┌────────────────┬──────────────────────────────────────────┐
│ 领域            │ 主要变化                                 │
├────────────────┼──────────────────────────────────────────┤
│ 安全            │ Windows 守护进程加固、SSRF 防护加强      │
│ 插件系统        │ ClawHub 支持、SDK 升级                   │
│ 新平台          │ Apple Watch 伴侣应用、iOS APNs 推送       │
│ 性能            │ Gateway 启动优化、按需加载插件            │
│ 控制台          │ Sessions 页面重设计、WebChat 渲染改进    │
│ 记忆系统        │ Active Memory 插件                       │
│ 飞书            │ 主题会话线程修复、文档评论改进            │
│ 模型            │ Grok 4.3、MiniMax-M2.5-highspeed 支持    │
└────────────────┴──────────────────────────────────────────┘

从 2026.2.x 到 2026.5.6 是一个安全 + 架构层面的重大跨越，建议尽快更新。

---

*升级完成时间：2026-05-07 深夜*
