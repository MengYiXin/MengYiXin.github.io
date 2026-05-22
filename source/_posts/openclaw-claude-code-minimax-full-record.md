---
title: "OpenClaw × Claude Code × MiniMax：我的 AI 工具全家桶升级全记录"
date: 2026-05-26 00:30:00
tags: [OpenClaw, Claude Code, MiniMax, AI工具, 飞书机器人, 龙虾, AI工作流, 智能体]
categories: [AI工具, 技术折腾]
---

![全家桶整体架构图](/images/ai-tools-stack-architecture.svg)

![三次升级踩坑时间线](/images/openclaw-upgrade-timeline.svg)

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

目标很明确：**把 OpenClaw 也切换到 MiniMax**，和 Hermes、Claude Code 用同一套 API key，同一个计费体系。

---

## 第一阶段：升级 OpenClaw 本体（2026.5.7 → 2026.5.6）

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

## 第二阶段：修改配置，切换 Provider（2026.5.7）

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

一开始我尝试加了 `apiKeyEnv: "ANTHROPIC_API_KEY"` 指向环境变量，**但 OpenClaw 不吃这个字段**，反而导致启动报错。实际测试发现 OpenClaw 会自动从环境变量 `ANTHROPIC_API_KEY` 读 key，不需额外配置。

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

## 第五阶段：第二次升级——2026.5.20（关键：gateway.cmd 版本号）

5 月 20 日，OpenClaw 又发新版本了。照例 `npm install -g openclaw --ignore-scripts`，版本升上去了，但 Gateway 重启后飞书没响应。

### 症状：Gateway 启动正常，但飞书 bot 无响应

```bash
$ openclaw status
Gateway: reachable ✅
Version: 2026.5.20 ✅
Feishu: not connected ❌
```

### 根因：gateway.cmd 里的版本号没更新

OpenClaw 在 Windows 上通过计划任务启动 Gateway，任务调用的脚本是 `gateway.cmd`。**升级后必须手动改这个文件里的版本号**，否则计划任务调用的还是旧版本程序。

```batch
# C:\Users\<用户>\AppData\Roaming\npm\node_modules\openclaw\gateway.cmd
# 升级后需要把这两行改成新版版本号：
set OPENCLAW_SERVICE_VERSION=2026.5.20
set OPENCLAW_SERVICE_PATH=%~dp0node_modules\openclaw\dist\service.js
```

### 正确升级步骤（Windows）

```bash
# 1. npm 升级
npm install -g openclaw --ignore-scripts

# 2. 查找新版 gateway.cmd 位置
dir /s /b %USERPROFILE%\AppData\Roaming\npm\gateway.cmd

# 3. 编辑新版 gateway.cmd，确认版本号匹配
#    找到 set OPENCLAW_SERVICE_VERSION=2026.5.20 这一行

# 4. 重启计划任务
schtasks /run /tn "OpenClaw Gateway"
```

---

## 第六阶段：第三次升级——2026.5.22（关键：base_url 路径错误导致 401）

5 月 22 日，OpenClaw 再次升级到最新版。这次更隐蔽——Gateway 启动正常，飞书也连上了，但 Hermes Agent 调用 MiniMax 时开始频繁报 401。

### 症状：401 invalid_request，curl 正常但 Agent 异常

```
Error: 401 Invalid request
hierarchyError: Unauthorized
```

同时 OpenClaw Gateway 的日志里也出现了同样的 401。

### curl 测试：网络完全正常

```bash
curl -X POST "https://api.minimaxi.com/anthropic/v1/messages" \
  -H "Authorization: Bearer $ANTHROPIC_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"MiniMax-M2.7","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'
```

✅ 返回正常。说明 key 本身没问题，网络也没问题。

### 根因：auxiliary_client.py 的端点判断逻辑

问题出在 OpenClaw 的 `auxiliary_client.py` 里。这个文件负责路由非对话请求（比如 vision 相关的 `/v1/coding_plan/vlm`）。

它的 `_endpoint_speaks_anthropic_messages()` 方法会**根据 base_url 路径判断这个端点是否走 Anthropic 格式**。

判断逻辑：

```python
def _endpoint_speaks_anthropic_messages(self, endpoint: str) -> bool:
    # 如果 base_url 包含 "/anthropic"，则认为是 Anthropic 端点
    return "/anthropic" in endpoint
```

问题就在这里：**当 base_url 被错误地设为 `https://api.minimaxi.com/anthropic`（带 `/anthropic` 后缀）时**，OpenClaw 认为这个端点是 Anthropic 端点，请求被发往：

```
https://api.minimaxi.com/anthropic/chat/completions
```

而不是正确的：

```
https://api.minimaxi.com/anthropic/v1/messages
```

同时，OpenClaw 在构造请求头时，`Authorization: Bearer <token>` 里的 token 变成了 Python 的字符串 `"None"`（因为它从 `os.environ.get("ANTHROPIC_API_KEY", None)` 读取，拿到的是字符串 `"None"` 而非 None）。

### 解决：base_url 统一改为 `/v1` 后缀

在 openclaw.json 的 minimax provider 里，把 baseUrl 从 `/anthropic` 改成 `/v1`：

```json
"providers": {
  "minimax": {
    "baseUrl": "https://api.minimaxi.com/v1",
    "api": "anthropic-messages",
    ...
  }
}
```

同时 Vision 部分的 base_url 也需要对应改为 `/v1`，否则 vision 端点会 404。

---

## 最终配置完整一览（2026.5.22 更新版）

`~/.openclaw/openclaw.json` 核心改动：

```json
{
  "meta": {
    "lastTouchedVersion": "2026.5.22",
    "lastTouchedAt": "2026-05-22T00:00:00.000Z"
  },
  "providers": {
    "minimax": {
      "baseUrl": "https://api.minimaxi.com/v1",
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
│ OpenClaw 版本   │ 2026.5.22 ✅                          │
│ 默认模型        │ MiniMax-M2.7 ✅                       │
│ API 类型        │ anthropic-messages ✅                 │
│ baseUrl        │ https://api.minimaxi.com/v1 ✅        │
│ 飞书连接        │ 正常 ✅                               │
│ Gateway         │ ws://127.0.0.1:18789 ✅              │
│ 身份            │ 影子 🖤 AI 助手 ✅                  │
└────────────────┴──────────────────────────────────────┘

三套 AI 工具现在全部统一用 MiniMax-M2.7：

┌────────────────┬──────────────────┬────────────────┐
│ 工具            │ 模型              │ 用途            │
├────────────────┼──────────────────┼────────────────┤
│ Hermes Agent   │ MiniMax-M2.7      │ 主力 AI 助手    │
│ Claude Code    │ MiniMax-M2.7      │ 本地代码助手    │
│ OpenClaw       │ MiniMax-M2.7      │ 飞书 AI 机器人  │
└────────────────┴──────────────────┴────────────────┘

---

## 踩坑完整记录（3次升级，累计9个坑）

┌────┬──────────────────────────┬──────────────────────────────┐
│ #  │ 问题                     │ 解决                         │
├────┼──────────────────────────┼──────────────────────────────┤
│ 1  │ npm 全局安装权限报错      │ --ignore-scripts 跳过安装脚本 │
│    │ preinstall 脚本调用 node  │ npm uninstall → install      │
│    │ 但 PATH 里 node 找不到    │                              │
├────┼──────────────────────────┼──────────────────────────────┤
│ 2  │ HTTP 404 model_not_found │ API 类型从 openai-completions │
│    │ MiniMax curl 通但        │ 改为 anthropic-messages      │
│    │ OpenClaw 内部路由 404   │                              │
├────┼──────────────────────────┼──────────────────────────────┤
│ 3  │ apiKeyEnv 字段不被识别   │ 删掉该字段，OpenClaw 自动读   │
│    │ 导致启动异常             │ 环境变量 ANTHROPIC_API_KEY   │
├────┼──────────────────────────┼──────────────────────────────┤
│ 4  │ 飞书配对全部失效         │ openclaw pairing approve      │
│    │ 升级后需重新配对          │ feishu S8HKK9NV              │
├────┼──────────────────────────┼──────────────────────────────┤
│ 5  │ Gateway 端口被旧进程锁定 │ powershell Stop-Process 强杀  │
│    │ PID 6708 占用 18789      │ 再重新启动 gateway            │
├────┼──────────────────────────┼──────────────────────────────┤
│ 6  │ 2026.5.20 升级后飞书     │ 手动改 gateway.cmd 里的       │
│    │ 无响应，Gateway 看起来   │ OPENCLAW_SERVICE_VERSION     │
│    │ 正常但 bot 不工作        │                              │
├────┼──────────────────────────┼──────────────────────────────┤
│ 7  │ 401 invalid_request      │ baseUrl 从 /anthropic 改为    │
│    │ curl 正常但 Agent 报 401 │ /v1，修复 auxiliary_client   │
│    │                          │ 路径判断                      │
├────┼──────────────────────────┼──────────────────────────────┤
│ 8  │ auxiliary_client vision  │ Vision base_url 改为 /v1    │
│    │ 端点 404，base_url 不一致 │ model 改为 MiniMax-M2       │
├────┼──────────────────────────┼──────────────────────────────┤
│ 9  │ Authorization header     │ 统一 base_url 路径后          │
│    │ token 变成字符串 "None"  │ os.environ.get 取值恢复正常   │
└────┴──────────────────────────┴──────────────────────────────┘

---

## Claude Code 接入 MiniMax

### 为什么也要接 Claude Code

本地写代码的时候，我习惯开 Claude Code 做代码助手。OpenClaw 管飞书里的 AI 对话，Claude Code 管本地 IDE 里的代码任务。两个工具各司其职，统一用 MiniMax 方便管理 API 费用。

### 接入方式

Claude Code 接入 MiniMax 也是通过 `ANTHROPIC_API_KEY` 环境变量。MiniMax 的 Anthropic 兼容端点 `https://api.minimaxi.com/v1` 对 Claude Code 来说就是一个普通的 Anthropic API 代理。

在 Windows 上，设置环境变量：

```batch
setx ANTHROPIC_API_KEY "sk-xxxxxxxxxxxxxxxx"
```

### 验证接入

```bash
$ claude -v
MiniMax-M2.7 connected ✅
```

---

## Claude Code 的 6 个核心 Skill

Claude Code 本身支持 Skill 扩展，按任务分工调用，稳定性翻倍。这 6 个是我日常最常用的：

┌────────────────┬─────────────────────────────────────────────┐
│ Skill 名称       │ 作用                                        │
├────────────────┼─────────────────────────────────────────────┤
│ explore         │ 通读整个项目代码，画出架构图，梳理数据流向    │
│ debugger        │ 定位报错位置，给出修复方案，带根因分析        │
│ code-review     │ 查漏洞、边界条件、风险点，适合 PR 合并前审查  │
│ test-engineer   │ 判断关键测试点，设计测试用例，覆盖率分析     │
│ code-simplifier │ 删除冗余代码，规范结构，改善可读性           │
│ security-review │ 扫描登录鉴权、权限边界、SQL注入等安全风险    │
└────────────────┴─────────────────────────────────────────────┘

### 安装方式

Skill 有三种安装位置：

**个人全局（所有项目通用）：**

```
Windows: %USERPROFILE%\.claude\skills\
Mac/Linux: ~/.claude/skills/
```

**项目本地（仅当前项目）：**

```
项目根目录/.claude/skills/
```

**插件市场（官方维护的 Skills）：**

```bash
# 终端命令安装
claude plugin install <技能名>@<市场名>

# 或在 Claude Code 内执行
/plugin marketplace add anthropics/skills
/plugin browse
# 然后选择要安装的 Skill
```

### 使用方式

安装完 6 个 Skill 后，按任务类型直接调用：

```bash
# 接手新项目，先通读架构
claude --skill explore

# 代码报错，找根因
claude --skill debugger

# 合并 PR 前安全审查
claude --skill code-review

# 设计测试用例
claude --skill test-engineer

# 清理冗余代码
claude --skill code-simplifier

# 安全风险扫描
claude --skill security-review
```

按职责分工调用，比一个 Agent 包揽所有任务稳定得多。

---

## OpenClaw 的 8 个核心 Skill（ClawHub 插件市场）

OpenClaw 的 Skill 系统通过 ClawHub 管理，这 8 个是常用插件：

┌───────────────────────┬──────────────────────────────────────┐
│ Skill 名称             │ 作用                                  │
├───────────────────────┼──────────────────────────────────────┤
│ agent-browser         │ 让 AI Agent 操作浏览器，做网页自动化    │
│ bb-browser            │ 浏览器辅助工具，强化网页交互能力        │
│ opencli               │ 命令行界面集成，增强终端操作能力        │
│ skill-vetter           │ 审核 Skill 质量，检查写法是否规范      │
│ self-improving-agent  │ 自我改进型 Agent，自动反思和优化输出   │
│ lossless-claw          │ 无损抓取插件，保持页面原始结构         │
│ control-center         │ 控制中心，管理多个 Agent 的调度和状态  │
│ openclaw-backup        │ 配置和数据的备份/恢复，保护工作环境    │
└───────────────────────┴──────────────────────────────────────┘

### 安装方式

**方式一：插件市场安装（推荐）**

```bash
# 在 OpenClaw 内执行
/plugin marketplace add anthropics/skills
/plugin browse
# 找到对应 Skill 安装

# 或终端命令
openclaw plugin install <技能名>
```

**方式二：手动安装**

下载 Skill 目录到 `~/.openclaw/skills/` 下。

### 安装后验证

```bash
openclaw skills list
```

看到所有 8 个 Skill 在列表里，就说明安装成功了。

---

## 关于计费

OpenClaw、Claude Code、Hermes Agent 三者本身都不管理计费，它们只是调用层。MiniMax 的实际计费方式（按量还是包月套餐）在 MiniMax 账户层面设置。

统一到 MiniMax 的核心价值：**三端同一套 API key，账单可预期，方便管理**，不用在三个地方分别充值。

---

## 附：OpenClaw 2026.2.x → 2026.5.22 主要变化

┌────────────────┬──────────────────────────────────────────┐
│ 领域            │ 主要变化                                  │
├────────────────┼──────────────────────────────────────────┤
│ 安全            │ Windows 守护进程加固、SSRF 防护加强       │
│ 插件系统        │ ClawHub 支持、SDK 升级                    │
│ 新平台          │ Apple Watch 伴侣应用、iOS APNs 推送        │
│ 性能            │ Gateway 启动优化、按需加载插件             │
│ 控制台          │ Sessions 页面重设计、WebChat 渲染改进      │
│ 记忆系统        │ Active Memory 插件                        │
│ 飞书            │ 主题会话线程修复、文档评论改进             │
│ 模型            │ Grok 4.3、MiniMax-M2.5-highspeed 支持    │
└────────────────┴──────────────────────────────────────────┘

---

*全文升级完成时间：2026-05-22 | 三次升级，累计 9 个坑，全部填平*
