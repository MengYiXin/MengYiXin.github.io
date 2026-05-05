---
title: "Hermes Agent 进阶：用 Skill 把日常工作流变成可复用工具"
date: 2026-05-05 16:00:00
tags:
  - Hermes Agent
  - Skill
  - 工作流
  - 自动化
categories: [AI工具]
---


Hermes Agent 有一个我很喜欢的设计：可以把任何重复性的工作流打包成 Skill，之后类似的事情只需要调用 Skill，AI 就会按既定流程执行，不用每次重新描述需求。

这篇文章总结一下我目前建的那些 Skill，以及背后的设计思路。

## 为什么需要Skill

不用 Skill 的时候，每次做类似的事情要：
1. 重新描述背景
2. 重新说明需求
3. 重新走一遍流程

有了 Skill，AI 直接加载对应的步骤和上下文，效率高很多。

## 我目前的Skill体系

![配图](https://raw.githubusercontent.com/MengYiXin/blog-images/main/04-hermes-skills-methodology.svg)

### blog-deployer — 博客自动部署

```
触发词：发布文章、更新博客、部署、提交博客
流程：
  Step 1：生成 md 文件（英文slug，放入 source/_posts/）
  Step 2：git add + commit + push 到 source 分支
  Step 3：等待 GitHub Actions 构建（约2-3分钟）
  Step 4：验证部署是否成功
```

关键设计点：
- 强制英文文件名（中文路径GitHub Pages会404）
- CNAME 必须显式复制（Actions不会自动带）
- 包含常见坑的说明（菜单不显示、评论失效等）

### git-proxy-heal — git代理自动修复

```
触发词：git连不上、push失败、代理问题
流程：
  诊断 → 判断是哪级失败 → 自动切换到可用路径
```

这个 Skill 核心不是执行固定步骤，而是**带分支逻辑的条件流程**：根据网络状态自动选择走 ghproxy、Clash 代理还是直连。

### session-backup — 记忆备份轮询

```
触发词：备份、session备份、记忆备份
架构：
  cron job 1：每5分钟检查哈希变化，变了就备份
  cron job 2：每分钟检查消息边界，每20条触发备份
  结果：~/.hermes/sessions_backups/ 最多30份，自动清理最旧的
```

这个 Skill 完全是自动化运行的，不需要人工触发。

### education-data-analysis — 教育数据对比

```
触发词：对比大学、哪个更容易、录取分数省份对比
输出格式：box-drawing字符表格，不用markdown
数据源：软科排名 + 高考志愿数据
```

设计重点是**输出格式标准化**：不管对比什么，始终用 box-drawing 字符画表格，先呈现数据再给简短结论。

### gstack — QA浏览器自动化

```
触发词：打开页面、截图、测试网站
功能：命令行控制Chromium完成页面交互、截图表单填写
坑：WSL root运行需加 --no-sandbox（已内置自动处理）
```

## Skill设计的一些原则

1. **触发词要具体**：不要只写"发布内容"，写"发布文章、更新博客、部署"，覆盖真实使用场景
2. **坑要提前说明**：把踩过的坑记在 Skill 里，下次不踩同样的坑
3. **条件分支要覆盖全**：特别是网络、认证类操作，要考虑各种失败情况
4. **输出格式要统一**：Skill 内部的输出格式要有规范，AI 调用时结果一致

## 下一步

打算建的 Skill：
- content-pipeline：批量内容生产流水线（小红书、知乎、公众号多平台适配文案）
- cold-email-sequencer：冷邮件序列生成与发送
- info-arbitrage-monitor：信息差监控（域名变动、关键词资讯）

核心思路是：**把副业/变现过程中重复的事情逐渐 Skill 化，让 AI 帮我干活，而不是每次重新对话**。

有兴趣交流的同学欢迎留言，或者直接到我的博客 mengyx.com.cn 找联系方式。
