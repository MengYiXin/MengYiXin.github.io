---
title: "Claude Code / Hermes 配置笔记：网络问题、代理配置与常见错误修复"
date: 2026-05-05 15:00:00
tags:
  - Claude Code
  - Hermes
  - 网络
  - 代理
  - WSL2
  - 故障排除
categories: [技术实战]
---

 AI 工具配置笔记：网络、代理与故障排除

**记录日期：2026-05-05**

---

## 一、网络配置核心问题

![配图](https://raw.githubusercontent.com/MengYiXin/blog-images/main/07-ai-tools-config-notes.svg)

### 1.1 GitHub 连接问题

**现象：** git push / clone 失败，Connection reset 或超时

**排查流程：**
```
1. 直接重试（不等3-5秒）
   ↓ 仍然失败
2. 检查网络稳定性（curl github.com）
   ↓ 仍失败
3. 配置代理
```

**代理配置（Clash）：**
```bash
# 查找 Clash 配置
find ~/.config/clash* -name "config.yaml"

# 获取端口（通常是 7890）
grep "mixed-port:" ~/.config/clash*/config.yaml

# 配置 git 代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

**关键经验：**
- GitHub 直连通常可行，不需要默认开代理
- curl 和 git 失败规律不同，属于网络波动
- 先重试一次，确认持续失败再配置代理

### 1.2 WSL2 网络问题

**现象：** gstack browse goto 超时

**原因：** WSL2 网络配置问题，非 gstack 本身问题

**解决：** 在 Windows 端使用 Chrome CDP，而不是 WSL 内的 Chromium

---

## 二、Clash 代理配置

### 2.1 配置文件位置

```
~/.config/clash*/config.yaml
```

### 2.2 常用端口

| 端口类型 | 默认值 | 用途 |
|----------|--------|------|
| mixed-port | 7890 | HTTP/HTTPS 代理 |
| port | 7890 | HTTP 代理 |
| socks-port | 7891 | SOCKS5 代理 |

### 2.3 git proxy 清理

```bash
# 查看当前配置
git config --global --get-regexp proxy

# 清除配置
git config --global --unset http.proxy
git config --global --unset https.proxy
```

---

## 三、Hermes 配置陷阱

### 3.1 base_url 陷阱

**错误写法：**
```yaml
minimax:
  api_key: sk-xxx
  base_url: https://api.minimaxi.com/anthropic/v1  # 多加了 /v1
```

**正确写法：**
```yaml
minimax:
  api_key: sk-xxx
  base_url: https://api.minimaxi.com/anthropic
```

### 3.2 API Key 位置陷阱

**错误写法：**
```yaml
providers:
  minimax:
    api_key: sk-xxx  # 错：写在 providers 下

model:
  provider: providers  # 错：provider 不能是 "providers"
```

**正确写法：**
```yaml
model:
  provider: minimax

minimax:
  api_key: sk-xxx
  base_url: https://api.minimaxi.com/anthropic
```

---

## 四、Python 环境问题

### 4.1 externally-managed-environment

**原因：** Debian/Ubuntu 系统的 PEP 668 保护

**解决方案：**

方案1（推荐）：加 `--break-system-packages` 参数
```bash
pip install xxx --break-system-packages
```

方案2：使用虚拟环境内的 pip
```bash
~/.hermes/hermes-agent/venv/bin/pip install xxx
```

### 4.2 Python 版本问题

**问题：** 提示 "Python 3.11 not found"

**解决：** 用 uv 安装指定版本
```bash
uv python list  # 查看可用版本
uv python install 3.11
```

---

## 五、WSL2 特殊问题

### 5.1 Chromium 沙箱问题

**错误：** Running as root without --no-sandbox is not supported

**原因：** Chromium 默认拒绝 root 用户使用沙箱

**解决：** 设置环境变量
```typescript
if (process.env.CI || process.env.CONTAINER || process.env.WSL_DISTRO_NAME) {
  launchArgs.push('--no-sandbox');
}
```

### 5.2 LD_LIBRARY_PATH 问题

**错误：** Chromium: error while loading shared libraries

**解决：** 设置 LD_LIBRARY_PATH
```json
"env": {
  "LD_LIBRARY_PATH": "/root/.local/lib/chromium-deps/usr/lib/x86_64-linux-gnu"
}
```

### 5.3 路径映射

WSL 和 Windows 路径映射：
```
WSL:  /root/.claude/skills/gstack
Win:  C:\Users\16080\.claude\skills\gstack
```

---

## 六、常见错误汇总

| 错误信息 | 原因 | 解决方案 |
|----------|------|----------|
| command not found: hermes | PATH 未配置 | 添加 ~/.local/bin 到 PATH |
| 401 authentication error | API key 错误 | 检查 config.yaml |
| externally-managed-environment | PEP 668 保护 | 加 --break-system-packages |
| --no-sandbox required | root 用户 | 设置 WSL_DISTRO_NAME |
| Connection reset | 网络波动 | 等几秒重试 |
| Unable to access GitHub | 代理/网络 | 配置 git proxy |

---

## 七、快速修复命令

```bash
# 1. hermes 命令找不到
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc

# 2. pip 安装失败
pip install xxx --break-system-packages

# 3. git push 失败（配置代理）
git config --global http.proxy http://127.0.0.1:7890

# 4. 清理 git 代理
git config --global --unset http.proxy && git config --global --unset https.proxy

# 5. 检查 Clash 端口
grep "mixed-port:" ~/.config/clash*/config.yaml
```

---

**记录于 2026-05-05**
