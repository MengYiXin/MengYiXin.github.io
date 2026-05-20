---
title: "给 OpenClaw 装了 14 个技能包，效率直接翻倍"
date: 2026-05-20 11:30:00
tags: [OpenClaw, AI工具, 效率, skill]
categories: [AI工具]
---

5月的某天，给 OpenClaw 做了一次技能大扩容。一口气装了 14 个 skills，覆盖浏览器自动化、安全扫描、代码审查、对话记忆、自我改进……装完跑了一周，体验是**真的回不去了**。

## 先说结论

OpenClaw 本身已经很强了，但装完这批 skills 之后，它从一个「能干的助手」变成了「能干且知道怎么干得更漂亮的助手」。每个 skill 各管一块，减少重复劳动，效果是实实在在的。

## 装了哪些

### 效率工具组

**bb-browser**
命令行刷 B站、知乎、微博、Twitter。不用开浏览器，直接在终端里抓内容、查热搜、批量获取数据。36 个平台覆盖，适合需要频繁获取社交媒体数据的场景。

**openclaw-agent-browser**
官方出的浏览器自动化 skill。填表、点按钮、截图、爬数据这些操作，装完它之后 OpenClaw 就能控制浏览器替你做。

**opencli-skill**
另一个命令行操作平台的 skill，更专注单个平台的内容获取和交互（B站、YouTube、雪球股票等）。有点安全警告，涉及外部 API 调用，用的时候稍加注意。

### 安全扫描组

**skill-vetter**
每个新 skill 安装之前先过一遍安检。扫代码查有没有密钥泄露、eval 调用、危险权限这些可疑操作。装第三方 skill 之前必跑，相当于先看体检报告再决定装不装。

### 学习记忆组

**self-improving-agent**
错误记录器。踩过的坑、下次不该再犯的错，它会自动记下来。长期学习型 skill，越用越懂你。

**lossless-claw**
对话蒸馏机。把聊天记录提炼出重点——谁是你、你关注什么、你做过什么决定。不是什么都记，是把噪声去掉，记住真正重要的。

### 系统运维组

**openclaw-control-center**
可视化控制面板。查看系统状态、AI 运行指标、管理定时任务。不是命令行了，给整个操作体验加了个 Dashboard。

**openclaw-backup**
定期备份 OpenClaw 所有数据（配置、技能、记忆），支持一键恢复。升级前跑一下，心里踏实。

### 代码开发组（6个自建）

这些是按 OpenClaw 的标准工作流拆出来的专项技能：

| Skill | 用途 |
|-------|------|
| explore | 通读项目、画架构图 |
| debugger | 定位报错、给修复方案 |
| code-review | 查漏洞、边界、风险 |
| test-engineer | 判断关键测试点 |
| code-simplifier | 删除冗余、规范结构 |
| security-review | 扫登录权限安全边界 |

之前这些能力混在一个 prompt 里，现在拆成独立 skill，每个只干一件事，OpenClaw 调用的时候更精准。

## 怎么装的

OpenClaw 的 skill 安装走 `hermes agent skill install` 命令。装完之后 OpenClaw 自动识别，下次对话直接调用。

```bash
hermes agent skill install bb-browser
hermes agent skill install skill-vetter
# ……以此类推
```

自建的 6 个 skill 放在 `~/.claude/skills/` 目录下，按标准 SKILL.md 格式写好就行。

## 用了两周的感受

最大的感受是：**信任感建立起来了**。

以前让 AI 读项目代码，总是担心它读不对、读不全。现在有 `explore` 专门负责通读，`code-review` 专门负责安全，`debugger` 专门负责报错……每个环节都有专项工具，AI 的发挥更稳定，我的复查成本也低了。

其次是 `lossless-claw` 和 `self-improving-agent` 这两个记忆类 skill，让 AI 对我的了解越来越深。不是每次新对话都要重新解释「我是谁」「我做什么」，它自己会记住。

## 适合谁

如果你用 OpenClaw（或类似的 AI coding agent）做正经工作，而不是偶尔尝鲜，这套 skills 值得装。不需要全部，挑自己最痛的那块先装——

- 经常爬数据 → 先装 bb-browser
- 经常装各种 skill → 先装 skill-vetter
- 同时开很多项目 → 先装 explore + lossless-claw

按需来，不用一次全装。

---

*有问题或者想聊具体某个 skill 的用法，群里见。*
