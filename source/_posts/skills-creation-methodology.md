---
title: "Claude Code Skills 创作方法论：7个实战Skills详解与设计思路"
date: 2026-05-05 15:00:00
tags:
  - Claude Code
  - Skill
  - 自动化
  - 工作流
categories: [技术实战]
---

 Skills 创作方法论：从重复操作到自动化

**记录日期：2026-05-05**

---

## 一、为什么要创建 Skills

### 1.1 问题背景

日常工作中经常处理重复性的技术工作：
- 重装 Hermes Agent 环境
- 配置 Git 代理
- 部署博客到 GitHub Pages
- 撰写冷邮件序列

每次遇到相同任务，都要重新回忆步骤、重新配置。这是时间浪费。

### 1.2 Skills 的价值

Skills 机制让人能够：
- **一句话触发复杂流程**：说"帮我重装 hermes"，Claude 自动完成所有步骤
- **跨 Session 记忆**：上一次配置的偏好，下一次自动应用
- **团队共享**：把经验固化成可复用的工具

### 1.3 Skills 哲学

```
简单任务 → 直接执行
重复任务 → 创建 Skill
重要任务 → 先测试再固化
```

---

## 二、Skills 创建实战

![配图](https://raw.githubusercontent.com/MengYiXin/blog-images/main/06-skills-creation-methodology.svg)

### 2.1 01-hermes-install

**触发词：** "install hermes", "reinstall hermes", "hermes broken"

**解决问题：** Hermes 重装时忘记步骤、配置出错

**内容结构：**

```
1. 前置检查
   - WSL2 是否可用
   - uv 是否安装
   - 网络是否正常

2. 安装步骤
   - 官方安装脚本
   - venv setup
   - API 配置（含 base_url 陷阱提醒）

3. 验证
   - hermes --version
   - hermes config edit

4. 常见问题
   - command not found → PATH 配置
   - 401 error → API key 检查
   - externally-managed → --break-system-packages
```

**关键细节：**
- MiniMax API 的 base_url 是 `https://api.minimaxi.com/anthropic`，不带 `/v1`
- Python 3.11 指定版本，避免系统 Python 版本不一致
- 软链接创建在 `~/.local/bin/hermes`

### 2.2 02-gstack-install

**触发词：** "install gstack", "gstack not working", "fix gstack browser"

**解决问题：** WSL2 环境下 gstack 的 Chromium 沙箱问题

**核心补丁：**

browser-manager.ts 需要添加 WSL_DISTRO_NAME 检测：
```typescript
if (process.env.CI || process.env.CONTAINER || process.env.WSL_DISTRO_NAME) {
  launchArgs.push('--no-sandbox');
}
```

**为什么需要这个补丁：**
- WSL2 是 root 用户运行
- Chromium 默认拒绝 root 用户使用沙箱
- 需要显式添加 --no-sandbox 参数

**LD_LIBRARY_PATH 配置：**
```json
"env": {
  "LD_LIBRARY_PATH": "/root/.local/lib/chromium-deps/usr/lib/x86_64-linux-gnu"
}
```

解决 Chromium 依赖库找不到的问题。

### 2.3 03-git-proxy-heal

**触发词：** "git push 失败", "git proxy", "can't connect to github"

**解决问题：** Git 操作失败时反复问用户选什么代理

**关键设计原则：**
- **不要问用户选择题**
- 自动查找 Clash 配置文件
- 直接获取 mixed-port
- 立即配置并重试

**实现逻辑：**
```bash
# 1. 查找 Clash 配置
find ~/.config/clash* -name "config.yaml" 2>/dev/null

# 2. 提取端口
grep "mixed-port:" config.yaml | awk '{print $2}'

# 3. 配置 git 代理
git config --global http.proxy http://127.0.0.1:$PORT
git config --global https.proxy http://127.0.0.1:$PORT

# 4. 重试操作
git push
```

**经验积累：**
- GitHub 直连通常可行，不需要默认开代理
- curl 能通时 git 可能失败，属于网络波动
- 先等几秒重试，确认持续失败再配置代理

---

## 三、Skills 文件格式

### 3.1 SKILL.md 结构

每个 Skill 目录包含：
```
skill-name/
├── SKILL.md          # 主文件（必需）
├── README.md         # 说明文档（可选）
└── assets/           # 资源文件（可选）
```

### 3.2 SKILL.md 格式

```markdown
---
name: skill-name
description: 一句话描述这个 Skill 做什么
triggers:
  - "触发词1"
  - "触发词2"
---

# Skill Name

## 功能说明
详细描述这个 Skill 的功能。

## 前置条件
- 条件1
- 条件2

## 使用方法
1. 步骤1
2. 步骤2

## 注意事项
- 注意1
- 注意2
```

---

## 四、Skills 创建流程

### 4.1 判断是否需要创建 Skill

```
这个操作会出现几次？
├── 1次 → 直接执行，不创建
├── 2-3次 → 考虑创建
└── 3次以上 → 一定要创建
```

### 4.2 创建步骤

```
1. 命名
   - 用数字前缀排序（01-hermes-install）
   - 简短易记
   - 用动词开头（install, fix, deploy）

2. 定义触发词
   - 覆盖正常场景（"install hermes"）
   - 覆盖错误场景（"hermes broken"）
   - 包含同义表达

3. 编写内容
   - 前置条件检查
   - 核心步骤（带验证点）
   - 常见问题处理

4. 测试
   - 用触发词激活
   - 完整执行一遍
   - 修复发现的问题

5. 发布
   - 推送到 GitHub（可选）
   - 更新记忆文件
```

---

## 五、高级技巧

### 5.1 触发词优化

**问题：** 触发词太具体，不容易记住

**解决：** 多个触发词覆盖同一 Skill
```yaml
triggers:
  - "install hermes"
  - "reinstall hermes"
  - "hermes broken"
  - "hermes not working"
```

### 5.2 条件分支

Skill 内容中可以使用条件：
```markdown
## 如果 uv 已安装
直接跳到克隆步骤

## 如果 uv 未安装
先安装 uv：curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 5.3 跨 Skill 引用

引用其他 Skill：
```markdown
如果需要配置代理，先运行 [[git-proxy-heal]]
```

---

## 六、已创建的 Skills 清单

| # | 名称 | 触发词数 | 用途 |
|---|------|----------|------|
| 01 | hermes-install | 3 | Hermes 重装 |
| 02 | gstack-install | 3 | gstack WSL2 安装 |
| 03 | git-proxy-heal | 3 | Git 代理自愈 |
| 04 | blog-deployer | - | GitHub Pages 部署 |
| 05 | cold-email-sequencer | - | 冷邮件序列 |
| 06 | content-pipeline | - | AI 内容生产 |
| 07 | info-arbitrage-monitor | - | 信息差监控 |

---

## 七、Skills 创作 Checklist

创建新的 Skill 前的检查清单：

```
□ 这个操作会重复发生吗？
□ 是否有多个触发词覆盖？
□ 前置条件是否明确？
□ 步骤是否有验证点？
□ 常见问题是否覆盖？
□ 是否测试过完整流程？
```

---

**记录于 2026-05-05**
