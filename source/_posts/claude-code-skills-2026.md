---
title: "Claude Code 装技能包：把 AI coding agent 调教成专属搭档"
date: 2026-05-20 12:00:00
tags: [Claude Code, AI工具, 效率, 编程助手]
categories: [AI工具]
---

OpenClaw 装完技能包之后体验飞升，顺手也给 Claude Code 来了一套。这两个工具定位不太一样：OpenClaw 更像一个全能助手，Claude Code 则更偏向编程侧。把合适的 skills 装到对的工具上，效率叠加效果比想象中更好。

## 为什么给 Claude Code 也装

Claude Code 本身能力很强，但默认状态下每次新对话都要重新对齐上下文。有些重复性的检查工作（安全扫漏洞、代码审查、错误定位），如果每次都要手把手教 AI 怎么做，积累不了经验。

装完 skills 之后，Claude Code 对项目结构越来越熟，对我的偏好越来越了解。不是每次都要从零开始，而是真的在「成长」。

## 装了哪些

### 自建 6 个专项 skill

这 6 个是按 Claude Code 的标准工作流拆出来的，各管一块：

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

**skill-vetter**
安全扫描。装任何第三方 skill 之前先跑一遍，检查有没有恶意代码、密钥泄露、危险权限。Claude Code 装 skill 很方便，但方便也意味着风险——这个工具就是来把门的。

**self-improving-agent**
错误笔记本。踩过的坑自动记录，下次遇到同类问题直接调取。不是聊天记录的那种「记住」，是真的把教训提炼成可复用的规则。

**lossless-claw**
对话蒸馏。把每次对话里的上下文、决策、偏好提炼出来存入长期记忆。Claude Code 跑的项目多，这个 skill 让它记住哪个项目是什么技术栈、有哪些特殊约束。

**openclaw-backup**
备份。Claude Code 的项目配置、skills、记忆文件定期备份，防止换环境或升级之后丢失积累。

**openclaw-control-center**
控制面板。查看 Claude Code 的运行状态、skill 调用记录、任务进度。不是日常必需，但调试问题的时候很好使。

## 安装过程踩了个坑

给 Claude Code 装 skill 和给 OpenClaw 装不太一样。OpenClaw 用 `hermes agent skill install` 命令直接装，Claude Code 的 skill 放在 `~/.claude/skills/` 目录下，需要手动放文件然后按标准格式写 SKILL.md。

写 SKILL.md 的时候要注意：

1. **name 字段**必须和目录名一致
2. **description** 要写得具体，Claude Code 是靠 description 决定要不要调用这个 skill 的
3. **触发条件**（usage_hint）要写清楚什么时候该用

skill-vetter 装的时候报了个安全警告，原因是它会扫描第三方 skill 的代码——这是设计如此，不是误报。用 `--force` 强装就行。

## 用下来的区别

给 OpenClaw 装 skill 和给 Claude Code 装 skill，感受不太一样：

OpenClaw 是「全能型」，skills 让它更好地调度各种工具链；
Claude Code 是「专精型」，skills 让它对编程这件事越来越内行。

两个工具现在互相补充：
- 需要多平台操作、查资料、写作 → OpenClaw
- 需要写代码、code review、debug → Claude Code

两边共享同一套记忆 skills，所以不管用哪个，背景上下文不用重复解释。

## 适合谁

如果你同时用多个 AI coding agent（比如 OpenClaw + Claude Code），建议两边都装，但装的 skill 可以不一样——按场景分配，让每个工具干它最擅长的事。

如果只用一个工具，那就按自己最痛的地方挑 skill 装：

- 经常多人协作项目 → 先装 security-review
- 项目经常出 bug → 先装 debugger
- 经常接新项目 → 先装 explore

---

*有具体安装问题或想聊某个 skill 怎么用的，欢迎群里交流。*
