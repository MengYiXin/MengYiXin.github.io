---
title: "给 OpenClaw 和 Claude Code 装上技能包，把 AI 助手当真正的搭档用"
date: 2026-05-20 14:00:00
tags: [OpenClaw, Claude Code, AI工具, 效率, skill]
categories: [AI工具]
---

5月的某天，给 OpenClaw 和 Claude Code 各做了一次技能大扩容。两个工具加起来装了 14 个 skills，覆盖浏览器自动化、安全扫描、代码审查、对话记忆、自我改进……装完跑了一周，体验是**真的回不去了**。

## 先说结论

AI coding agent 本身已经很强了，但装完这批 skills 之后，它从一个「能干的助手」变成了「能干且知道怎么干得更漂亮的助手」。每个 skill 各管一块，减少重复劳动，效果是实实在在的。

两个工具定位不太一样：OpenClaw 更像一个全能助手，Claude Code 则更偏向编程侧。把合适的 skills 装到对的工具上，效率叠加效果比想象中更好。

## OpenClaw 装了哪些

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

## Claude Code 装了哪些

### 自建 6 个专项 skill

按 Claude Code 的标准工作流拆出来的专项技能，各管一块：

| Skill | 用途 |
|-------|------|
| explore | 通读项目结构、画架构图 |
| debugger | 定位报错、分析调用链 |
| code-review | 查漏洞、边界、风险点 |
| test-engineer | 划测试优先级、找等价类 |
| code-simplifier | 删重复代码、拆过长函数 |
| security-review | 扫登录鉴权安全边界 |

之前这些能力全塞在一个 system prompt 里，AI 有时候会漏掉某个环节。现在拆成独立 skill，Claude Code 知道什么时候该调哪个。

### 从 clawhub 装的

**skill-vetter** — 安全扫描。装任何第三方 skill 之前先跑一遍，检查有没有恶意代码、密钥泄露、危险权限。

**self-improving-agent** — 错误笔记本。踩过的坑自动记录，下次遇到同类问题直接调取。

**lossless-claw** — 对话蒸馏。把每次对话里的上下文、决策、偏好提炼出来存入长期记忆。Claude Code 跑的项目多，这个 skill 让它记住哪个项目是什么技术栈、有哪些特殊约束。

**openclaw-backup** — 备份。定期备份项目配置、skills、记忆文件，防止换环境或升级之后丢失积累。

**openclaw-control-center** — 控制面板。查看运行状态、skill 调用记录、任务进度。调试问题的时候很好使。

## 安装过程踩了个坑

给 Claude Code 装 skill 和给 OpenClaw 装不太一样。OpenClaw 用 `hermes agent skill install` 命令直接装，Claude Code 的 skill 放在 `~/.claude/skills/` 目录下，需要手动放文件然后按标准格式写 SKILL.md。

写 SKILL.md 的时候要注意：

1. **name 字段**必须和目录名一致
2. **description** 要写得具体，Claude Code 是靠 description 决定要不要调用这个 skill 的
3. **触发条件**（usage_hint）要写清楚什么时候该用

skill-vetter 装的时候报了个安全警告，原因是它会扫描第三方 skill 的代码——这是设计如此，不是误报。用 `--force` 强装就行。

## 用下来的感受

最大的感受是：**信任感建立起来了**。

以前让 AI 读项目代码，总是担心它读不对、读不全。现在有 `explore` 专门负责通读，`code-review` 专门负责安全，`debugger` 专门负责报错……每个环节都有专项工具，AI 的发挥更稳定，复查成本也低了。

两个工具现在互相补充：
- 需要多平台操作、查资料、写作 → OpenClaw
- 需要写代码、code review、debug → Claude Code

两边共享同一套记忆 skills，所以不管用哪个，背景上下文不用重复解释。

## 适合谁

如果你同时用多个 AI coding agent（比如 OpenClaw + Claude Code），建议两边都装，但装的 skill 可以不一样——按场景分配，让每个工具干它最擅长的事。

如果只用一个工具，那就按自己最痛的地方挑 skill 装：

- 经常爬数据 → 先装 bb-browser
- 经常装各种 skill → 先装 skill-vetter
- 同时开很多项目 → 先装 explore + lossless-claw
- 项目经常出 bug → 先装 debugger
- 经常多人协作项目 → 先装 security-review

按需来，不用一次全装。

---

*有问题或者想聊具体某个 skill 的用法，群里见。*
