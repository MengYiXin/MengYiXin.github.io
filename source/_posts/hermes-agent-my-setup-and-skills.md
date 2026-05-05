---
title: "Hermes Agent 使用手记：从安装到打造个人AI工作流"
date: 2026-05-05 16:00:00
tags:
  - Hermes Agent
  - AI工具
  - 工作流
  - 笔记
categories: [AI工具]
---


前阵子把 Hermes Agent 在我的 WSL2 环境里跑起来了，顺便配了一系列 Skill，把日常的信息处理、博客发布、Git 管理都串进了这个 AI 工作流。今天把这段经历整理出来，供想入坑的同学参考。

## 什么是Hermes Agent

Hermes Agent 是 [Nous Research](https://nousresearch.com/) 开发的一个开源 AI 智能体框架（MIT 许可证）。它不是普通的聊天机器人，而是一个"住在你服务器上的自主 AI 助手"——24小时在线，有记忆，会学习，用得越多越懂你。

核心指标：

┌───────────────┬────────────┐
│ GitHub Stars  │ 27,000+    │
│ 内置工具      │ 47+        │
│ 支持平台      │ 12+        │
│ MCP 可接入    │ 6,000+     │
│ 最低部署成本  │ $5/月      │
│ 内存占用      │ <500MB     │
│ 许可证        │ MIT        │
└───────────────┴────────────┘

一句话概括：**Hermes 是第一个出厂就带"缰绳"的 Agent，而且缰绳会自己长大。**

## 我的环境

- 系统：WSL2（Windows下的Linux子系统）
- 用户目录：/mnt/f/（文件放F盘，空间大）
- 博客：Hexo + GitHub Pages，自定义域名 mengyx.com.cn
- GitHub 仓库：MengYiXin/MengYiXin.github.io（source分支存Hexo源文件，master存HTML）
- 主要使用场景：AI副业/变现工作流、信息差赚钱、内容批量生产

## 我都配了哪些Skill

Skill 是 Hermes 的核心扩展机制——你可以把任何重复性的工作流打包成一个可复用的技能。

下面是我目前在建/已建的几个核心 Skill：

### 1. blog-deployer
博客自动化部署。Hexo源文件在 source 分支，push 到 GitHub 后 Actions 自动 build 并发布到 master 分支。关键点：

- 文件名必须用英文 slug（中文文件名在 GitHub Pages 上会404）
- CNAME 文件必须叫 `CNAME`（不是 CNAME.txt）
- deploy.yml 里要显式 `cp source/CNAME _deploy/CNAME`，否则自定义域名会失效
- 评论系统原来用的 Valine（依赖LeanCloud），已官方宣布关闭服务，建议换 Twikoo

### 2. git-proxy-heal
WSL 下 Clash 代理导致 git push/pull 失败的自动修复。

症状：`Failed to connect to localhost port 7890`，或者 curl https://github.com 卡住。

我的解法是 **~/bin/git wrapper 三级自动切换**：

┌──────────────────────────────────────────────────────┐
│ ~/bin/git wrapper 三级 fallback                       │
├──────────────────────────────────────────────────────┤
│ 级1：ghproxy 通（经Clash检测） → 走 ghproxy 镜像     │
│ 级2：ghproxy 挂 + Clash 在线   → 走 Clash 代理直连   │
│ 级3：Clash 也挂了              → 直连                │
└──────────────────────────────────────────────────────┘

每次 git 命令动态判断，全程无需手动干预。

### 3. session-backup
Session 备份轮询系统。两条 cron job 驱动：

| job_id          | schedule         | 触发条件          │
|-----------------│-----------------│-------------------|
| fee5e5957c9e    | */5 * * * *     | 哈希变化时触发    │
| 5dd74c629478    | * * * * *       | 每分钟检查消息边界 |

- 每 20 条消息触发一次备份（可调）
- 最多保留 30 份，超出自动清理最旧的
- 备份存放：`~/.hermes/sessions_backups/`

### 4. education-data-analysis
跨省高校/高考数据对比分析。用 box-drawing 字符画表格，数据源用软科排名（shanghairanking.cn）和高考志愿填报数据。

### 5. gstack
gstack 是 Hermes 生态里的 QA 浏览器工具，可以用命令行控制 Chromium，完成页面截图表单填写等操作。在 WSL 下有个已知坑：root 用户运行会报 `Running as root without --no-sandbox`，解法是运行前 `export CONTAINER=1`，已内置自动处理。

## 写在最后

用 Hermes Agent 一段时间下来，最直接的感受是：**它把 AI 从"问完就走"的工具变成了"帮我干活"的助手**。配好 Skill 之后，很多重复性的事情——发布博客、查数据、备份记忆——都能自动跑起来。

下一步打算把更多日常工作流 Skill 化，特别是内容批量生产和信息差监控方向。有兴趣的同学可以关注我的博客 mengyx.com.cn，后续会陆续分享实战细节。
