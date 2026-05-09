---
title: "Hermes Agent 飞书双向通信实战：从诊断到开机自启"
date: 2023-05-09 11:30:00
tags: [Hermes Agent, 飞书, 自动化, AI工作流, 系统集成]
categories: [AI工具, 技术折腾]
---

## 背景：昨晚还能用，今天怎么就断了？

我的 AI 工作流里，Hermes Agent 是主力助手，OpenClaw 是飞书 bot。昨天晚上把 Hermes 接入了飞书，实现了：

```
飞书 → Hermes：发消息，AI 回答 ✅
Hermes → 飞书：任务完成主动推送结果 ✅
```

今早起来，飞书 bot 完全没有响应。发消息给 Hermes，零回复。

目标很明确：**修好双向通信，顺便做成开机自启**。

---

## 第一阶段：诊断问题根因

### 症状

飞书给 Hermes 发消息，Hermes 不回答。但之前那条"飞书通知测试"的消息是能收到的，说明 Hermes → 飞书推送是通的，飞书 → Hermes 接收断了。

### Step 1：确认 Hermes Agent 本身在跑

```bash
$ ps aux | grep hermes
root  264  ... /usr/local/lib/hermes-agent/venv/bin/python3 /root/.local/bin/hermes
root 1239  ... hermes -p side-hustle chat
root 1361  ... hermes -p huakun chat
root 1971  ... hermes -p master chat
```

✅ Hermes Agent 进程都在，4个 profile 都活着。

### Step 2：查 Hermes Gateway 状态

```bash
$ hermes gateway status
✗ Gateway is not running
```

**找到了！ Hermes Gateway 没在跑。**

Hermes Agent 和 Hermes Gateway 是两个东西：
- **Hermes Agent**：AI 助手的大脑，跑在云端/本地
- **Hermes Gateway**：消息网关，负责接入飞书/Telegram/Discord 等平台，是消息进出的大门

大门关着，消息自然进不来。

---

## 第二阶段：启动 Gateway

### 启动命令

```bash
tmux new -d -s hermes-gateway 'hermes gateway run'
```

### 验证状态

```bash
$ hermes gateway status
✓ Gateway is running (PID: 3815)
  (Running manually, not as a system service)
WSL note:
  The gateway is running in foreground/manual mode (recommended for WSL).
  Use tmux or screen for persistence across terminal closes.
```

### 看日志确认飞书连上

```bash
$ tail -30 ~/.hermes/logs/gateway.log
2026-05-09 11:02:53,309 INFO gateway.run: Starting Hermes Gateway...
2026-05-09 11:02:53,310 INFO gateway.run: Session storage: /root/.hermes/sessions
2026-05-09 11:02:56,593 INFO gateway.run: Connecting to feishu...
2026-05-09 11:02:58,556 INFO gateway.platforms.feishu: [Feishu] Connected in websocket mode (feishu)
2026-05-09 11:02:58,567 INFO gateway.run: ✓ feishu connected
2026-05-09 11:02:58,576 INFO gateway.run: Gateway running with 1 platform(s)
```

✅ 飞书 WebSocket 长连接建立成功！

### 验证双向通信

立即在飞书发一条消息给 Hermes：

```
你好
```

```
回复 yixin: 
你好 yixin！有什么需要帮忙的吗？
```

✅ **双向通信完全恢复。**

---

## 第三阶段：排查为什么 Gateway 会停

Gateway 今天没跑，说明上次 WSL 重启后它没有自动起来。需要做成开机自启。

### 尝试方案：systemd user service

先写了一个 systemd service 文件：

```ini
[Unit]
Description=Hermes Gateway (Feishu/Webhook)
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/lib/hermes-agent/venv/bin/hermes gateway run
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

放到 `~/.config/systemd/user/hermes-gateway.service`。

### 遇到问题：systemd 不可用

```bash
$ systemctl --user daemon-reload
# 第一次尝试：WSL 环境还没完全初始化，user-mode systemd 还没就绪
# 报错：Failed to connect to bus
```

等了几秒再试，systemd 突然可用了：

```bash
$ systemctl --user daemon-reload
$ systemctl --user enable hermes-gateway
$ systemctl --user start hermes-gateway
```

### systemd 接管后的状态

```bash
$ hermes gateway status
● hermes-gateway.service - Hermes Agent Gateway
     Loaded: loaded (...hermes-gateway.service; enabled)
     Active: active (running) since Sat 2026-05-09 11:07:24 CST
   Main PID: 4039 (python)
```

### 开启 linger（允许服务在用户登出后继续运行）

```bash
$ sudo loginctl enable-linger root
```

---

## 第四阶段：WSL 重启后 systemd 服务的持久性问题

虽然 systemd 服务跑起来了，但 **WSL 重启后 systemd 会被重置**，linger 的效果不一定能跨 WSL 重启保持。

所以还需要 Windows 侧的开机自启作为保底。

### Windows 开机自启：Startup 文件夹方案

把启动脚本放到 Windows 启动文件夹：

```
C:\Users\16080\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\
```

启动脚本内容：

```bat
@echo off
wsl.exe -d Ubuntu -e bash -c "tmux kill-session -t hermes-gateway 2>/dev/null; tmux new -d -s hermes-gateway 'hermes gateway run' 2>&1"
```

### 为什么用 batch 脚本而不是直接 WSL 命令

WSL 的 UNC 路径问题（`\\wsl.localhost\...`）在 CMD 里处理复杂。batch 脚本先 `cd /d C:\Users\16080` 切换到本地盘符路径，避免 CMD 坚持要用 UNC 的坑。

---

## 最终架构

```
Windows 开机
    ↓
Startup 文件夹 → 运行 .bat
    ↓
WSL bash → tmux new -d -s hermes-gateway
    ↓
tmux session: hermes-gateway
    ↓
hermes gateway run（systemd 托管）
    ↓
飞书 WebSocket 长连接（wss://msg-frontier.feishu.cn）
    ↓
Hermes Agent 处理消息
```

```
┌──────────────────────────────────────────────────────────────────────┐
│                      Hermes × 飞书 双向通信架构                       │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   飞书用户  ──WebSocket──▶  Hermes Gateway ──API──▶  Hermes Agent    │
│                        (消息入口)              (AI 推理)              │
│                                                                      │
│   Hermes Agent  ──API──▶  Hermes Gateway  ──HTTP──▶  飞书用户       │
│                       (消息出口)               (推送)                │
│                                                                      │
│   ┌──────────────────┐    ┌──────────────────┐    ┌─────────────┐ │
│   │  Hermes Agent     │    │  Hermes Gateway  │    │  飞书       │ │
│   │  /usr/local/bin/  │◀──▶│  tmux session    │◀──▶│  WebSocket  │ │
│   │  hermes           │    │  hermes-gateway  │    │  wss://...  │ │
│   └──────────────────┘    └──────────────────┘    └─────────────┘ │
│                                                                      │
│   自启动链条：                                                    │
│   Windows Startup ──▶ .bat ──▶ WSL tmux ──▶ hermes gateway run     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 踩坑完整记录

```
┌────┬────────────────────────────────────────┬──────────────────────────────────┐
│ #  │ 问题                                    │ 解决                              │
├────┼────────────────────────────────────────┼──────────────────────────────────┤
│ 1  │ 飞书发消息 Hermes 不回                  │ Hermes Gateway 未启动             │
│    │ hermes gateway status 返回 not running  │ tmux new -d -s hermes-gateway    │
├────┼────────────────────────────────────────┼──────────────────────────────────┤
│ 2  │ systemctl --user 报错                 │ WSL 环境未就绪，等几秒再试        │
│    │ "Failed to connect to bus"             │ systemd 实际上是可用的            │
├────┼────────────────────────────────────────┼──────────────────────────────────┤
│ 3  │ Gateway 和 tmux session 冲突           │ systemd 和 tmux 同时跑会互相阻止  │
│    │ "Gateway already running (PID 3815)"    │ 先 kill 掉 tmux 再启动 systemd   │
├────┼────────────────────────────────────────┼──────────────────────────────────┤
│ 4  │ WSL 重启后 systemd 被重置              │ Windows Startup 文件夹保底        │
│    │ linger 跨重启 不一定生效                │ .bat → WSL tmux → hermes run     │
├────┼────────────────────────────────────────┼──────────────────────────────────┤
│ 5  │ CMD 不支持 UNC 路径                    │ cd /d C:\Users\xxx 切换本地路径   │
│    │ CMD.EXE UNC path not supported         │ 再执行 wsl.exe                   │
└────┴────────────────────────────────────────┴──────────────────────────────────┘
```

---

## 关键命令汇总

```bash
# 查 Gateway 状态
hermes gateway status

# 启动 Gateway（tmux 方式）
tmux new -d -s hermes-gateway 'hermes gateway run'

# systemd 管理
systemctl --user daemon-reload
systemctl --user enable hermes-gateway
systemctl --user start hermes-gateway
systemctl --user status hermes-gateway

# 开启 linger
sudo loginctl enable-linger root

# 看 Gateway 日志
tail -50 ~/.hermes/logs/gateway.log

# 确认飞书连接
hermes gateway status
# 期望输出: ✓ feishu connected
```

---

## 总结

昨晚能通、今早断了的根本原因：**Hermes Gateway 没有设开机自启**。

解决方案分两层：

1. **systemd user service**：日常保活，WSL 开机后自动拉起
2. **Windows Startup 文件夹**：WSL 重启后的保底方案，双保险

现在状态：
- 飞书双向通信：✅ 正常
- 开机自启：✅ 配置完成
- tmux session `hermes-gateway`：✅ 运行中

---

*修复完成时间：2026-05-09 上午*
