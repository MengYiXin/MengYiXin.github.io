---
title: 远程给家用 Windows 电脑安装 Hermes Agent：SSH 连接与踩坑全记录
date: 2026-05-11 00:30:00
categories:
  - AI工具
  - 技术记录
tags:
  - Hermes Agent
  - SSH
  - Windows
  - 远程控制
  - Python
  - MiniMax
---

# 远程给家用 Windows 电脑安装 Hermes Agent：SSH 连接与踩坑全记录

## 背景

目标是**远程连接家里/公司的 Windows 电脑，通过 SSH 在上面自动化安装 Hermes Agent**，不需要有人在那台电脑前操作。整个过程分两大部分：

1. **远程连接 Windows**（SSH + 密码认证）
2. **远程安装 Hermes**（无网络、无 GUI、全自动）

目标机器是一台联想台式机（品牌机自带的管理员账户），网络环境为局域网（同一 WiFi 或 VPN）。

---

{% raw %}
<figure>
  <figcaption>图1 · 远程连接全貌：本机 WSL → 局域网 SSH → Windows 目标机</figcaption>
  <iframe src="./figs/remote-windows-ssh-arch.html" width="100%" height="580" style="border:none;border-radius:8px;"></iframe>
</figure>
{% endraw %}

---

## 第一部分：远程连接 Windows

### 前置条件

Windows 10/11 专业版/家庭版都自带 OpenSSH 服务器功能，只需要开启即可。

#### 1.1 开启 Windows SSH 服务

在目标 Windows 机器上（需要管理员权限），以**管理员身份**打开 PowerShell，执行：

```powershell
# 安装 OpenSSH 服务器（如果还没有）
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 启动 SSH 服务并设为自动
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

安装完成后，Windows 会自动在防火墙里开放 22 端口。

#### 1.2 确认 SSH 连接

```bash
# 从本机（Linux/macOS/WSL）测试连接
sshpass -p '你的Windows密码' ssh -o StrictHostKeyChecking=no 用户名@IP "whoami"

# 例如（局域网内）
sshpass -p '[REDACTED]' ssh localadmin@192.168.1.138 "whoami"
# 成功返回：localadmin
```

> **为什么要用 sshpass？**
> 直接 `ssh user@ip` 会弹出交互式密码输入提示，自动化脚本里不方便。本机如果没有 sshpass，先装：
> ```bash
> apt-get install -y sshpass   # Debian/Ubuntu/WSL
> brew install sshpass         # macOS
> ```

#### 1.3 远程执行命令的三种方式

SSH 连接成功后，可以远程执行任何 Windows 命令。以下三种方式按场景选择：

**方式 A：直接命令（简单场景）**

```bash
sshpass -p '密码' ssh -o StrictHostKeyChecking=no localadmin@IP "cmd /c dir C:\Users"
```

**方式 B：PowerShell 脚本（推荐，脚本逻辑复杂时）**

```bash
# 写好 .ps1 脚本 → scp传过去 → 远程执行
sshpass -p '[REDACTED]' scp /tmp/check_disk.ps1 localadmin@192.168.1.138:C:/Users/localadmin/
sshpass -p '[REDACTED]' ssh localadmin@192.168.1.138 "powershell -ExecutionPolicy Bypass -File C:\Users\localadmin\check_disk.ps1"
```

**方式 C：Python 脚本（跨平台，最可靠）**

```bash
# 写好 .py 脚本 → scp传过去 → 远程执行
sshpass -p '[REDACTED]' scp /tmp/fix_auth.py localadmin@192.168.1.138:C:/Users/localadmin/
sshpass -p '[REDACTED]' ssh localadmin@192.168.1.138 "python C:\Users\localadmin\fix_auth.py"
```

> **重要经验：Windows SSH 中文编码问题**
> Windows SSH 服务端默认 GBK 编码，WSL/macOS 客户端用 UTF-8，直接执行 `cmd /c dir` 时中文文件名全部乱码。
> 解决方案：**永远不要让命令穿墙**。把脚本写到本地 → scp 传过去 → 远程执行。避免了编码问题，也方便调试。

---

## 第二部分：远程安装 Hermes Agent

### 完整流程

#### 2.1 第一步：确认连通性和 Python 环境

```bash
sshpass -p '[REDACTED]' ssh -o StrictHostKeyChecking=no localadmin@192.168.1.138 "python --version"
```

- 返回 `Python 3.12.x` → Python 已在 PATH，继续下一步
- 返回"不是内部或外部命令" → Python 不在 PATH，见 2.3 节

#### 2.2 第二步：停止拦截安装的安全服务

品牌机（联想/戴尔/惠普）通常预装了"系统更新保护"之类的安全服务，会拦截 Python 安装包静默安装。

```powershell
Stop-Service -Name EasiUpdate3Protect -Force
Stop-Service -Name EasiUpdate3 -Force
Stop-Service -Name EasiUpdate -Force
```

> **经验：** `sc delete` 需要比管理员更高权限，一般不成功。用 `Stop-Service -Force` 先停掉进程即可，服务停止后安装就能进行，不一定要删掉服务。

#### 2.3 第三步：找 Python 或安装 Python

Windows 远程机器上 Python 经常出现在以下位置（按优先级排序）：

1. `C:\Users\Lenovo\Downloads\python-*.exe` — **最优先，找到了直接用**，不需重新下载
2. `C:\Python312` — 可能是空目录（安装中断），勿被迷惑
3. 注册表 `HKLM:\Software\Python\PythonCore\3.x\InstallPath` 记录的位置

**搜索步骤（按顺序）：**

```powershell
# 1. 看 Downloads 目录有没有安装包
Get-ChildItem "C:\Users\Lenovo\Downloads\python*.exe" -ErrorAction SilentlyContinue

# 2. 验证 Python 是否真的在 PATH
python --version

# 3. 查注册表
Get-ItemProperty "HKLM:\Software\Python\PythonCore\3.12\InstallPath"
```

**安装 Python（如果没找到）：**

优先用 Downloads 目录里的安装包（已下载好，不走网络，最快）：

```cmd
C:\Users\Lenovo\Downloads\python-3.12.8-amd64.exe /quiet InstallAllUsers=0 PrependPath=1 Include_pip=1
```

等待 30 秒后验证：

```bash
sshpass -p '[REDACTED]' ssh localadmin@192.168.1.138 "python --version"
# 应返回 Python 3.12.x
```

> **坑：Downloads 里的安装包可能不完整**
> 如果只有几百 KB，说明下载中断了（遇到过 python-installer.exe 只有 13% 大小），需要找完整的安装包或重新下载完整版。

#### 2.4 第四步：安装 setuptools（Python embed 版必需）

通过非完整安装包装的 Python（或者 embed 版）默认没有 pip/setuptools，必须先装：

```bash
sshpass -p '[REDACTED]' ssh localadmin@192.168.1.138 "python -m pip install setuptools wheel"
```

#### 2.5 第五步：Clone 并打包 Hermes 源码传到目标机器

**关键经验：Windows 远程机器访问 GitHub 极慢/超时。** 解决方案是本机 clone 后打包传过去。

```bash
# 本机 clone（WSL/Linux）
git clone --depth=1 https://github.com/NousResearch/hermes-agent.git

# 打包成 .tar.gz（不能用 .tar，Windows 自带 tar 解不了 .tar.gz）
tar -czf hermes-agent.tar.gz hermes-agent

# 传到远程
sshpass -p '[REDACTED]' scp hermes-agent.tar.gz localadmin@192.168.1.138:C:/Users/localadmin/
```

**远程解压（用 Python 解压，Windows tar 解不了 .tar.gz）：**

```bash
sshpass -p '[REDACTED]' ssh localadmin@192.168.1.138 "python -c \"import tarfile; tarfile.open('C:\\Users\\localadmin\\hermes-agent.tar.gz').extractall('C:\\Users\\localadmin')\""
```

#### 2.6 第六步：pip install 本地源码

```bash
sshpass -p '[REDACTED]' ssh localadmin@192.168.1.138 "python -m pip install C:\\Users\\localadmin\\hermes-agent"
```

#### 2.7 第七步：配置 API Key

这里有**巨大的坑**，详细说明见下一节。先讲正确做法：

**方案 A：机器级环境变量（所有用户都能读到，推荐）**

```powershell
# 用 PowerShell 写到 Machine 级注册表，所有账户共享
[Environment]::SetEnvironmentVariable('ANTHROPIC_API_KEY', '你的API密钥', 'Machine')
[Environment]::SetEnvironmentVariable('ANTHROPIC_BASE_URL', 'https://api.minimaxi.com/anthropic', 'Machine')
```

**方案 B：setx /M（机器级，永久）**

```cmd
setx ANTHROPIC_API_KEY "你的API密钥" /M
setx ANTHROPIC_BASE_URL "https://api.minimaxi.com/anthropic" /M
```

> **不要只写 localadmin 的 .env 文件**——联想普通用户读不到 `C:\Users\localadmin\.hermes\.env`，必须用机器级写入。

#### 2.8 第八步：验证安装

```bash
sshpass -p '[REDACTED]' ssh localadmin@192.168.1.138 "hermes --version"
# 应返回：Hermes Agent v0.13.0
```

---

{% raw %}
<figure>
  <figcaption>图2 · Hermes 远程安装 8 步流程与关键命令</figcaption>
  <iframe src="./figs/remote-windows-install-flow.html" width="100%" height="620" style="border:none;border-radius:8px;"></iframe>
</figure>
{% endraw %}

---

## 踩坑全记录（经验精华）

### 坑 1：auth.json pool 名与 model.provider 不匹配（最常见）

**症状：** auth.json 正确、.env 正确、API Key 正确，但 Hermes 请求头是 `Authorization: Bearer None`，请求发到了错误的 `api.minimax.io`。

**根本原因：** hermes 根据 `config.yaml` 里 `model.provider` 的值从 auth.json 的 `credential_pool` 里找同名 pool：

- `model.provider: minimax` → 读 `credential_pool.minimax[]`
- `model.provider: anthropic` → 读 `credential_pool.anthropic[]`

本次案例中，config.yaml 设了 `provider: minimax`，但 auth.json 里 `minimax` pool 缓存了错误的 `.io` base_url（应为 `.com`）。`anthropic` pool 虽然 key/URL 都对，但根本不会被读到。

**解法（两步全做）：**

Step 1：修改 config.yaml，把 `model.provider` 统一成 `anthropic`

Step 2：重写 auth.json，只保留 `anthropic` pool，删掉 `minimax` pool

> **最容易漏的点：** auth.json 里的 `minimax` pool 的 `base_url` 经常被误配成 `https://api.minimax.io/anthropic`（.io 错），应为 `https://api.minimaxi.com/anthropic`（.com 对）。即使 key 完全正确，URL 错了依然 401。

---

### 坑 2：config.yaml 被误删成 0 字节

**症状：** auth.json 正确、.env 正确，但 Hermes 跑不起来或 401。

**根本原因：** 手动编辑 config.yaml 时替换逻辑未匹配到内容，文件被**清空为 0 字节**。之后 hermes 读 config.yaml 得到空内容。

**特征：** 文件存在但大小为 0：

```python
import os
print(os.path.getsize(r'C:\Users\Lenovo\.hermes\config.yaml'))
# 输出：0
```

**解法：直接删掉空的 config.yaml**，让 hermes 完全依赖 auth.json 和 .env：

```python
import os
os.remove(r'C:\Users\Lenovo\.hermes\config.yaml')
```

> **预防：** 不要手动编辑 .yaml 文件，用 `hermes config set` 命令写入配置。

---

### 坑 3：多用户下 API Key 读取位置问题（四层叠加）

**表象：** localadmin SSH 里 `hermes config show` 显示正确 key，但联想用户双击 Hermes 运行时仍 401，token prefix 不对（`sk-cp-Doz8BI` 而非 `sk-cp--7fKd`）。

**根本原因（四层叠加）：**

1. **读取位置不同**：hermes 读 `.env` 时用的是**运行用户的 %USERPROFILE%**
   - localadmin SSH：`%USERPROFILE%` = `C:\Users\localadmin` → 读到正确 key
   - 联想用户双击：`%USERPROFILE%` = `C:\Users\Lenovo` → `.hermes\.env` 可能为空

2. **config.yaml 结构歧义**：文件中可能同时存在 `providers.minimax`（正确）和独立根级 `minimax:` 块（旧版遗留），导致解析歧义

3. **auth.json 凭证池缓存**：401 后的 key 会被标记为 `exhausted` 并缓存，即使更新了 config.yaml，hermes 仍从 auth.json 读已缓存的 exhausted 凭证

**401 调试速查顺序：**

```
1. 检查 auth.json → 找 exhausted key + 检查 minimax pool 的 base_url
   ⚠️ minimax pool 的 base_url 经常被误配成 https://api.minimax.io（.io错）
   应为 https://api.minimaxi.com/anthropic（.com对）
2. 检查 .env → 看 ANTHROPIC_API_KEY 值对不对（两个横杠 sk-cp--）
3. 检查 config.yaml → providers.minimax.api_key 对不对
4. 查 model_catalog.enabled → 可能是 catalog 覆盖
5. 杀残留进程 → 新 key 不生效的常见原因
```

---

### 坑 4：Hermes 装在 localadmin AppData 里，联想用户没有 NTFS 权限

**症状：** `hermes.bat` 双击报错 `Access is denied`。

**根本原因：** `InstallAllUsers=0` 导致 hermes 只装进 `C:\Users\localadmin\AppData\...`，Lenovo 普通用户完全没有该目录的读取权限（NTFS 权限）。

**解法两步：**

**Step 1：给联想用户开 Python312 目录读+执行权限**

```powershell
icacls "C:\Users\localadmin\AppData\Local\Programs\Python\Python312" /grant "DESKTOP-0LVHJ7D\Lenovo:RX" /t
```

> 注意：用户名必须写死 `机器名\用户名` 格式（`DESKTOP-0LVHJ7D\Lenovo`），不能用 PowerShell 变量 `$user` 替换，否则 icacls 展开失败。

**Step 2：用计划任务启动 Hermes（不用 runas）**

runas 在 bat 里会导致密码框闪退，改用计划任务：

```powershell
# hermes_task.ps1
$action = New-ScheduledTaskAction -Execute "C:\Users\localadmin\AppData\Local\Programs\Python\Python312\Scripts\hermes.exe"
$trigger = New-ScheduledTaskTrigger -AtLogon
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable
Register-ScheduledTask -TaskName "HermesAgent" -Action $action -Trigger $trigger -Settings $settings -User "localadmin" -Password "[REDACTED]" -RunLevel Highest -Force
```

hermes.bat 只需一行触发命令（不需要密码）：

```bat
@echo off
chcp 65001 >nul
title Hermes Agent
schtasks /run /tn HermesAgent
```

---

### 坑 5：计划任务里 `python -m hermes` 会 No module named hermes

**症状：** 计划任务触发后日志显示 `No module named hermes`。

**原因：** hermes 安装在 localadmin 的 Scripts 目录，但 Python 模块搜索路径未配置正确。

**解法：直接用 hermes.exe 完整路径作为 Execute，不走 python -m：**

```powershell
# 正确
$action = New-ScheduledTaskAction -Execute "C:\Users\localadmin\AppData\Local\Programs\Python\Python312\Scripts\hermes.exe"

# 错误（会失败）
$action = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c python -m hermes"
```

---

### 坑 6：远程 PowerShell 命令执行成功但输出为空

**症状：** `ssh ... "python -c \"...\" "` 执行成功（exit code 0）但完全没有输出，难以调试。

**根本原因：** Windows SSH 服务端默认 GBK 编码，WSL SSH 客户端用 UTF-8，两端编码不一致导致输出被吞。

**解法：把 Python 脚本写到本地 → scp 传过去 → 远程执行：**

```bash
# 写脚本到 /tmp/fix.py（本地）
# scp 传到远程
sshpass -p '[REDACTED]' scp /tmp/fix.py localadmin@192.168.1.138:C:/Users/localadmin/fix.py
# 远程执行
sshpass -p '[REDACTED]' ssh localadmin@192.168.1.138 "python C:\Users\localadmin\fix.py"
```

> **教训：** 涉及读写文件、调试输出时，始终用 Python 脚本 + scp 方案，不要 direct command inline。

---

## 总结

整个过程踩了无数坑，但核心经验就三条：

1. **Windows SSH 编码问题**：永远不要让复杂命令穿墙，本地写脚本 → scp → 远程执行
2. **多用户 NTFS 权限**：软件装在 localadmin，但普通用户要用，必须用 icacls 开权限或用计划任务兜底
3. **auth.json 凭证池缓存**：认证问题的根源往往是缓存，调试顺序第一步永远是查 auth.json

完整踩坑记录和解决方案已整理成 Hermes Skill：**`windows-remote-hermes-install`**，可在 [我的 GitHub Skills 仓库](https://github.com/MengYiXin/my-skills) 查看。

---

## 连接信息速查

```
IP:           192.168.1.138
SSH 用户:     localadmin
SSH 密码:     [REDACTED]
RDP PIN:      [REDACTED]
机器名:       DESKTOP-0LVHJ7D
Hermes 路径:  C:\Users\localadmin\AppData\Local\Programs\Python\Python312\Scripts\hermes.exe
Python 路径:  C:\Users\localadmin\AppData\Local\Programs\Python\Python312\python.exe
```

---

*写于 2026-05-11，远程调试联想台式机时的血泪记录。*
