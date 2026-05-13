---
title: "记一次远程Windows飞书重启：SSH无法启动GUI的彻底解决"
date: 2026-05-13 09:45:00
tags: [SSH, Windows, PowerShell, 飞书, 远程运维]
categories: 技术
---

## 背景

有一台远程Windows电脑（192.168.1.138），飞书客户端卡死了，9个进程全部无响应。通过SSH（localadmin账号）远程连接，想把飞书重启一下。

结果发现：**SSH会话根本没办法启动GUI程序**。

这篇文章记录了我踩过的所有坑，以及最终怎么解决的。

---

## 环境信息

| 项目 | 内容 |
|---|---|
| 远程IP | 192.168.1.138 |
| SSH账号 | localadmin |
| 飞书进程名 | Feishu.exe |
| 飞书路径 | C:\Users\Lenovo\AppData\Local\Feishu\app\Feishu.exe |
| 飞书运行用户 | Lenovo（不是localadmin） |

---

## 所有失败的方法

### ❌ 方法1：SSH直接Start-Process

```bash
sshpass -p 'Dos70001' ssh localadmin@192.168.1.138 \
  "Start-Process 'C:\Users\Lenovo\AppData\Local\Feishu\app\Feishu.exe'"
```

**结果**：命令执行后没有任何报错，但飞书根本没起来。

**原因**：SSH会话是纯字符界面，没有Windows图形会话上下文。Start-Process找不到可以显示窗口的桌面会话。

---

### ❌ 方法2：runas切换到联想用户

```powershell
runas /user:DESKTOP-0LVHJ7D\Lenovo "C:\Users\Lenovo\AppData\Local\Feishu\app\Feishu.exe"
```

**结果**：报错，密码验证失败。

**原因**：runas需要目标用户的正确密码。SSH里无法交互输入密码，而且Lenovo账号的密码也不是qinyuan（那是PIN码）。

---

### ❌ 方法3：PowerShell Start-Process with PSCredential

```powershell
Start-Process -FilePath 'C:\Users\Lenovo\AppData\Local\Feishu\app\Feishu.exe' -Credential $cred
```

**结果**：报错，用户凭证不正确。

**原因**：localadmin账号无权以Lenovo用户身份启动进程。

---

### ❌ 方法4：Chrome浏览器打开飞书网页版

```bash
sshpass -p 'Dos70001' ssh localadmin@192.168.1.138 \
  "C:\Program Files\Google\Chrome\Application\chrome.exe https://feishu.cn"
```

**结果**：Chrome报GPU进程错误，无法启动。

**原因**：SSH会话没有图形环境，即使启动Chrome也找不到显示器，GPU进程初始化失败。

---

### ❌ 方法5：WSL里用win32yank调用Windows程序

```bash
win32yank -o | powershell -Command ...
```

**结果**：win32yank命令不存在。

**原因**：win32yank未安装在远程PC上，而且这个工具本身是用于WSL和Windows之间传递数据的，不适合用来启动GUI。

---

### ❌ 方法6：hermes chat -q 触发browser工具

```bash
hermes chat -q "用browser工具打开飞书：导航到 https://feishu.cn"
```

**结果**：

```
prompt_toolkit.output.win32.NoConsoleScreenBufferError:
No Windows console found. Are you running cmd.exe?
```

**原因**：SSH里hermes cli初始化时prompt_toolkit尝试访问Windows控制台，但SSH会话没有真实的控制台设备。这个问题和飞书无关，纯粹是hermes cli的终端检测逻辑在SSH里失效了。

---

### ❌ 方法7：xfreerdp远程桌面连接

```bash
xfreerdp /v:192.168.1.138 /u:localadmin /p:Dos70001
```

**结果**：RDP可以建立连接，但命令行只能建立连接，无法操作GUI。

**原因**：xfreerdp建立的是观察者模式的连接，无法远程控制鼠标键盘。

---

### ❌ 方法8：taskkill + 直接路径启动

```bash
# 先杀掉卡死的进程
taskkill /f /im Feishu.exe

# 再直接启动
sshpass -p 'Dos70001' ssh localadmin@192.168.1.138 \
  "'C:\Users\Lenovo\AppData\Local\Feishu\app\Feishu.exe'"
```

**结果**：进程杀掉后，飞书没有自动重启。手动调用路径无效。

**原因**：和Direct Start-Process一样，没有GUI会话承接这个进程。

---

## ✅ 最终成功的方法

### 核心思路

SSH本身是字符界面，无法直接启动GUI程序。但Windows的PowerShell Remoting（WinRM）可以跨会话执行命令——利用本地PowerShell Remoting，把启动命令注入到真实桌面会话执行。

### 步骤1：开启WinRM服务

```bash
sshpass -p 'Dos70001' ssh -o StrictHostKeyChecking=no localadmin@192.168.1.138 \
  "powershell -Command \"Start-Service WinRM; Get-Service WinRM | Select-Object Name,Status\""
```

输出：

```
Name    Status
----    ------
WinRM   Running
```

**关键**：WinRM服务必须是Running状态，否则后续的Invoke-Command会失败。

### 步骤2：通过PowerShell Remoting启动飞书

```bash
sshpass -p 'Dos70001' ssh -o StrictHostKeyChecking=no localadmin@192.168.1.138 \
  "powershell -Command \"Invoke-Command -ComputerName localhost -ScriptBlock { Start-Process 'C:\\Users\\Lenovo\\AppData\\Local\\Feishu\\app\\Feishu.exe' }\""
```

执行后返回空，没有报错。

### 步骤3：验证进程

```bash
sshpass -p 'Dos70001' ssh -o StrictHostKeyChecking=no localadmin@192.168.1.138 \
  "powershell -Command \"Get-Process Feishu -ErrorAction SilentlyContinue | Measure-Object | Select-Object -ExpandProperty Count\""
```

输出：**8**

飞书起来了，8个进程正常运行。

---

## 原理图解

```
┌─────────────────────────────────────────────┐
│  SSH Session (localadmin, 纯字符界面)         │
│  无GUI会话上下文                             │
└─────────────────┬───────────────────────────┘
                  │ SSH 文本通道
                  ▼
┌─────────────────────────────────────────────┐
│  PowerShell Remoting (WinRM)               │
│  -ComputerName localhost                    │
│  寻找本地真实桌面会话                        │
└─────────────────┬───────────────────────────┘
                  │ WinRM 通道
                  ▼
┌─────────────────────────────────────────────┐
│  ScriptBlock { Start-Process ... }          │
│  ← 在真实桌面会话(Lenovo用户)中执行          │
└─────────────────────────────────────────────┘
                  │
                  ▼
         ┌────────────────┐
         │ Feishu.exe启动 │
         │ 8个进程运行    │
         └────────────────┘
```

**关键点**：`Invoke-Command -ComputerName localhost` 走的是本地PowerShell Remoting通道，它会在当前机器上找一个真实的桌面会话来执行ScriptBlock，从而成功启动GUI程序。

---

## 完整一键脚本

```bash
#!/bin/bash
# restart_feishu_remote.sh
HOST="192.168.1.138"
USER="localadmin"
PASS="Dos70001"
FEISHU_PATH="C:\\Users\\Lenovo\\AppData\\Local\\Feishu\\app\\Feishu.exe"

# 杀掉现有卡死的飞书进程
sshpass -p "$PASS" ssh -o StrictHostKeyChecking=no "$USER@$HOST" \
  "powershell -Command \"Get-Process Feishu -ErrorAction SilentlyContinue | Stop-Process -Force\""

sleep 2

# 开启WinRM
sshpass -p "$PASS" ssh -o StrictHostKeyChecking=no "$USER@$HOST" \
  "powershell -Command \"Start-Service WinRM\""

# 通过PowerShell Remoting启动飞书
sshpass -p "$PASS" ssh -o StrictHostKeyChecking=no "$USER@$HOST" \
  "powershell -Command \"Invoke-Command -ComputerName localhost -ScriptBlock { Start-Process '$FEISHU_PATH' }\""

sleep 3

# 验证
COUNT=$(sshpass -p "$PASS" ssh -o StrictHostKeyChecking=no "$USER@$HOST" \
  "powershell -Command \"(Get-Process Feishu -ErrorAction SilentlyContinue | Measure-Object).Count\"")
echo "Feishu进程数: $COUNT"
```

---

## 总结

| 方法 | 结果 | 原因 |
|---|---|---|
| SSH + Start-Process | ❌ 失败 | SSH无GUI上下文 |
| runas切换用户 | ❌ 失败 | 密码错误 |
| PowerShell PSCredential | ❌ 失败 | localadmin无权以Lenovo身份启动 |
| Chrome打开网页版 | ❌ 失败 | 无显示器，GPU报错 |
| hermes chat -q | ❌ 失败 | prompt_toolkit无控制台 |
| xfreerdp | ❌ 失败 | 只读连接，无法操作 |
| **PowerShell Remoting** | ✅ **成功** | 注入真实桌面会话执行 |

**核心教训**：SSH会话和远程桌面会话是两个完全独立的上下文。SSH里执行的命令没有图形桌面，只有通过PowerShell Remoting这样的机制，才能把命令发送到真实桌面会话去执行。

---

*文章写于2026年5月13日，远程PC环境：Windows 10专业版，飞书v7.x*
