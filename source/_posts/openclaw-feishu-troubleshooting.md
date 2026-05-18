---
title: OpenClaw 飞书掉线排障：从 plugin duplicate 到 delivery pending 全记录
date: 2026-05-18 11:30:00
categories:
  - AI工具
  - 技术记录
tags:
  - OpenClaw
  - Feishu
  - Troubleshooting
  - WebSocket
  - Windows
---

# OpenClaw 飞书掉线排障：从 plugin duplicate 到 delivery pending 全记录

## 问题现象

飞书 bot 不回应了。重启 OpenClaw 后日志里出现两个关键警告：

```
Config warnings:
- plugins.entries.feishu: plugin feishu: duplicate plugin id detected;
  global plugin will be overridden by global plugin
  (C:\Users\16080\.openclaw\npm\node_modules\@openclaw\feishu\dist\index.js)
```

以及一个 delivery pending 一直重试失败：

```
[delivery-recovery] Delivery entry 053620cc-72e8-4726-b478-8f16c35cae6f
  delivery state is send_attempt_started; refusing blind replay without adapter reconciliation
[delivery-recovery] Retry failed for delivery 053620cc-72e8-4726-b478-8f16c35cae6f:
  delivery state is send_attempt_started; refusing blind replay...
```

同时飞书 API 请求返回 99991663 错误（应用权限问题）。

## 根因分析

### 问题1：Feishu plugin duplicate

OpenClaw 检测到两个同 ID 的 feishu 插件：

1. **全局插件**：`C:\Users\16080\.openclaw\npm\node_modules\@openclaw\feishu\dist\index.js`
2. **扩展插件**：`C:\Users\16080\.openclaw\extensions\feishu\index.ts`

两个插件注册了相同的 ID，global plugin 被 extension 覆盖，但没有正确初始化 WebSocket 连接。

### 问题2：delivery pending 卡住

OpenClaw 在尝试发送消息时网络中断（飞书 WebSocket 断开），导致 delivery entry 处于 `send_attempt_started` 状态。系统拒绝盲目重放，因为不知道 adapter 当前状态。

### 问题3：飞书 API 99991663

应用权限不足，可能原因：
- 应用只开通了「用户身份」权限，没开「应用身份」
- 或者 token 过期需要刷新

## 解决步骤

### Step 1：修复 plugin duplicate

编辑 `C:\Users\16080\.openclaw\openclaw.json`，把 extensions 里的 feishu 配置注释掉或删掉：

```json
{
  "plugins": {
    "entries": {
      // 注释或删除这里的 feishu 配置
      // "feishu": { ... }
    }
  }
}
```

重启 OpenClaw，duplicate warning 应该消失。

### Step 2：清理 pending delivery

pending delivery entry 会阻止新消息发送。进入 OpenClaw 后台（通常是 `http://127.0.0.1:18789`），找到 delivery queue，手动删除卡住的那条记录。

或者重启 OpenClaw（完整重启，不是 hot reload），delivery 队列会重新初始化。

### Step 3：验证飞书连接

查看日志确认：

```
[feishu] starting feishu[default] (mode: websocket)
[info]: [ 'client ready' ]
[gateway] ready
```

`client ready` 表示 WebSocket 已连接。如果只有 `starting` 没有 `ready`，说明连接失败了。

### Step 4：确认权限配置

飞书开放平台 → 应用 → 权限管理，确认同时开通了：
- `im:chat:readonly`（用户身份）
- `im:chat:readonly`（应用身份）

两个都要开，只开一个不够。

## 预防措施

1. **不要同时启用 npm 全局 feishu 插件和本地 extension 插件**——二选一
2. **网络中断后关注 delivery recovery 日志**——卡住超过 5 分钟需要手动干预
3. **定期重启 OpenClaw**——WebSocket 长连接有时候会静默断开

## 关键日志位置

```
C:\Users\16080\AppData\Local\Temp\openclaw\openclaw-2026-05-18.log
```

出现问题时先看这个文件，搜 `delivery` 或 `feishu` 定位根因。