---
title: "Claude Code 使用全记录：安装配置、Hermes、gstack 与网络修复笔记"
date: 2026-05-05 15:00:00
tags:
  - Claude Code
  - Hermes
  - gstack
  - 开发工具
  - WSL2
categories: [技术实战]
---

 Claude Code 使用全记录

**记录日期：2026-05-05**

---

## 一、与 Claude Code 的相遇

我叫蒙沂鑫，目前在 IT 行业从事解决方案相关工作。2026年4月开始探索 AI 编码工具，朋友推荐了 Claude Code，说这是目前最强的 AI 编程助手。经过这段时间的摸索，把安装配置全过程、使用技巧、遇到的问题和解决方案都记录下来，供自己回顾，也供有类似需求的朋友参考。

**我的技术背景：**
- 熟悉信创、算力、AI
- 常用 Windows + WSL2 Ubuntu
- 不喜欢折腾代理，希望开箱即用
- 目前使用 MiniMax-M2.7 作为主力模型（通过 Claude API）

---

## 二、Claude Code 安装过程

![配图](https://raw.githubusercontent.com/MengYiXin/blog-images/main/05-claude-code-full-guide.svg)

### 2.1 环境准备

环境信息：
- Windows 11 Home China
- WSL2 Ubuntu（通过 `wsl --install` 安装）
- Claude Code 版本：2.1.122

### 2.2 安装步骤

Claude Code 的安装非常简洁：

1. **下载安装包**
   从官网下载 Claude Code 安装程序

2. **完成基础配置**
   - 设置工作目录
   - 配置 API 访问（使用 MiniMax API）

3. **配置 CLAUDE.md**
   这是项目级别的行为指南：

```markdown
# CLAUDE.md
Behavioral guidelines to reduce common LLM coding mistakes.

## 1. Think Before Coding
- Don't assume. Don't hide confusion. Surface tradeoffs.
- State your assumptions explicitly. If uncertain, ask.

## 2. Simplicity First
- Minimum code that solves the problem. Nothing speculative.
- No features beyond what was asked.

## 3. Surgical Changes
- Touch only what you must. Clean up only your own mess.
- Match existing style, even if you'd do it differently.

## 4. Goal-Driven Execution
- Define success criteria. Loop until verified.
```

---

## 三、Hermes Agent 安装与配置

### 3.1 为什么用 Hermes

Hermes Agent 是一个基于 Claude 的 AI Agent，可以联网搜索、浏览网页、记忆管理。我在 2026-04-29 首次安装，后来在 2026-05-05 升级到 v0.12.0。

### 3.2 安装过程

**前提条件：**
- Windows 10/11，已开启 WSL2
- Ubuntu 已启动
- 网络正常（能访问 GitHub）

**一键安装命令：**
```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

这个脚本会自动完成：
- 安装 uv（Python 包管理器）
- 安装 Python 3.11
- 安装 Node.js 22
- 克隆代码仓库到 ~/.hermes/hermes-agent/
- 创建虚拟环境 venv
- 安装所有 Python 依赖
- 安装 npm 依赖
- 链接 hermes 命令到 ~/.local/bin/hermes

### 3.3 API Key 配置

安装完成后，必须配置 API Key：

```bash
hermes config edit
```

在 minimax 段落填入：
```yaml
minimax:
  api_key: 你的sk-xxx密钥
  base_url: https://api.minimaxi.com/anthropic
```

**重要陷阱：**
- base_url 不带 /v1 后缀
- API key 填在 minimax: 段落，不是 providers: 里

### 3.4 初始化配置

```bash
hermes setup
```

按提示选择：
- Provider：minimax
- Model：MiniMax-2.7

### 3.5 创建 Windows 桌面快捷方式

创建 `hermes.bat`：
```batch
@echo off
title Hermes Agent
wsl -d Ubuntu -e bash -c "cd /root && /root/.local/bin/hermes"
pause
```

双击即可启动。

### 3.6 Hermes 常用命令

| 命令 | 说明 |
|------|------|
| hermes | 启动对话 |
| hermes setup | 重新配置 |
| hermes config edit | 编辑配置文件 |
| hermes config set | 查看/修改单项配置 |
| hermes gateway | 启动消息网关 |
| hermes update | 更新到最新版本 |

### 3.7 升级记录

- 2026-04-29：首次安装 v0.11.0
- 2026-05-05：升级到 v0.12.0（ Autonomous Curator、自改进循环升级、4个新推理provider、Spotify+Google Meet集成）

---

## 四、gstack WSL2 安装

### 4.1 什么是 gstack

gstack 是一个浏览器自动化工具，可以控制 Chrome 浏览器进行网页操作。

### 4.2 安装过程

gstack 安装在 WSL2 环境下，路径映射：
- WSL：`/root/.claude/skills/gstack`
- Windows：`C:\Users\16080\.claude\skills\gstack`（符号链接）

### 4.3 关键配置

**browser-manager.ts 补丁：**
添加 `WSL_DISTRO_NAME` 到沙箱检测，解决 WSL2 root 用户 Chromium --no-sandbox 问题：

```typescript
if (process.env.CI || process.env.CONTAINER || process.env.WSL_DISTRO_NAME) {
  launchArgs.push('--no-sandbox');
}
```

**settings.json 配置：**
```json
"env": {
  "PATH": "/root/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
  "LD_LIBRARY_PATH": "/root/.local/lib/chromium-deps/usr/lib/x86_64-linux-gnu"
}
```

### 4.4 验证结果

- browse status → healthy
- browse url → about:blank（正常）
- browse goto → 超时（WSL网络配置问题，非gstack本身问题）

---

## 五、网络问题修复

### 5.1 问题描述

Git 操作经常失败，特别是 `git push` 到 GitHub。

### 5.2 代理配置

代理是 Clash（HTTP端口7890）。

**配置步骤：**
1. 查找 Clash 配置文件（`~/.config/clash*/config.yaml`）
2. 获取 mixed-port
3. 配置 git 代理：
```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

### 5.3 重要经验

**经验1：** GitHub 直连通常可行，不需要默认开代理。测试发现 curl 和 git clone 不开代理也能连上，之前超时是网络抖动。

**经验2：** 网络不稳定，curl 和 git 均有间歇性失败。curl 能通时 git 可能失败，反之亦然，属于网络波动而非代理问题。

**经验3：** 遇到 Connection reset 或超时，先等几秒重试，不要立即判定为代理问题。如果反复失败再配置代理。

### 5.4 网络恢复流程

```
网络操作失败 → 等3-5秒重试 → 检查Clash进程 → 启动Clash（端口7890）
```

---

## 六、Skills 创作

### 6.1 为什么要创建 Skills

Skills 是 Claude Code 的扩展机制，可以把重复操作自动化。需求场景：
- 日常重复操作做成 Skills
- 一句话触发复杂流程
- 跨 Session 记忆偏好

### 6.2 已创建的 Skills

| Skill | 触发词 | 用途 |
|-------|--------|------|
| 01-hermes-install | "install hermes", "reinstall hermes" | Hermes 重装（官方安装脚本、venv setup、MiniMax API 配置） |
| 02-gstack-install | "install gstack", "gstack not working" | gstack WSL2 安装（含 browser-manager.ts 补丁） |
| 03-git-proxy-heal | "git push 失败", "git proxy" | Git 代理自愈（自动查找 Clash 端口并配置） |
| 04-blog-deployer | - | GitHub Pages 博客一键部署 |
| 05-cold-email-sequencer | - | 冷邮件序列生成与发送自动化 |
| 06-content-pipeline | - | AI 内容批量生产（多平台适配） |
| 07-info-arbitrage-monitor | - | 信息差监控（域名/WHOIS/社媒/关键词） |

### 6.3 Skills 创建方法论

**触发词设计：**
- 覆盖同义表达（"install hermes" = "reinstall hermes"）
- 包含错误场景（"hermes broken"）
- 简短易记

**内容结构：**
1. 前置条件检查
2. 核心步骤（带验证）
3. 常见问题排查
4. 触发词列表

---

## 七、记忆系统

### 7.1 记忆类型

| 类型 | 用途 | 示例 |
|------|------|------|
| user | 用户背景信息 | 技术偏好 |
| feedback | 操作习惯偏好 | 代理配置、不喜欢问问题 |
| project | 当前项目状态 | 开发进度 |
| reference | 外部系统入口 | GitHub、IMA 知识库地址 |

### 7.2 记忆文件结构

每个记忆文件包含：
- `name`: 唯一标识
- `description`: 一行描述（用于判断相关性）
- `type`: 类型（user/feedback/project/reference）
- `content`: 具体内容

### 7.3 IMA 知识库

访问地址：https://ima.qq.com/wikis（独立 SPA 页面）

知识库内容：
- 微信精选文章
- 各类项目资料
- 技术文档
- 培训材料

---

## 八、已授权操作清单

为了减少重复确认，预先授权了以下操作类别：

### 8.1 GitHub 相关
- 创建新仓库并推送
- 克隆仓库到 `F:/Repos/MengYiXin/`
- 所有 git push/pull/clone 操作

### 8.2 开发工具安装
- npm 包安装（`npm install -g`）
- Python 包（pip install）
- Node.js / npx
- 构建命令（`npm run build`）

### 8.3 Skills 和插件
- 安装任何 skill
- 添加 MCP server
- 安装 superpowers 全家桶

### 8.4 网络访问
- WebSearch / WebFetch
- Chrome DevTools Protocol
- GitHub API
- 所有外部 URL

### 8.5 文件系统
- 读取桌面 PDF/Word/Excel 文件
- 整理桌面文件
- 创建/修改/删除本地文件

---

## 九、reporter-web 项目

### 9.1 项目概述

reporter-web 是周报日报生成器，基于 AI 的自动化工具。

### 9.2 技术栈

- React + TypeScript + TailwindCSS + Vite
- GitHub Pages 部署
- GitHub Gist 作为数据存储
- 月之暗面 Kimi API

### 9.3 功能模式

- **M 模式**：科技公司日报
- **Y 模式**：三江集团周报

### 9.4 部署流程

```bash
npm run build
git push
```

---

## 十、常见问题汇总

### Q1: hermes 命令找不到
**原因：** 安装后没有刷新 shell 环境，或 ~/.local/bin 不在 PATH 里
**解决：**
```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Q2: pip install 报错 externally-managed-environment
**原因：** Debian/Ubuntu 系统 Python 受 PEP 668 保护
**解决：**
```bash
pip install xxx --break-system-packages
```

### Q3: config.yaml 修改后不生效
**原因：** API key 应填在 minimax: 段落，而非写在 providers: 里
**解决：** 确保格式正确，冒号后有空格

### Q4: Git push 失败
**原因：** 网络问题或代理配置错误
**解决：** 先等3-5秒重试，确认持续失败后再配置代理

---

## 十一、桌面文件整理

Windows 桌面文件组织结构：

```
桌面/
├── 00-Hermes安装说明.txt
├── install_hermes.sh
├── hermes.bat
├── 01-文档/
├── 02-演示文稿/
├── 03-表格/
├── 04-快捷方式/
├── 05-参考资料/
├── Linux PPT/
└── ...（按项目分类）
```

---

## 十二、参考资料

- Claude Code 版本：2.1.122
- Hermes Agent 版本：v0.12.0
- Python：3.11.15
- OpenAI SDK：2.32.0
- Node.js：v22.16.0
- WSL2 Ubuntu

---

**记录于 2026-05-05**
