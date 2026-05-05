---
title: "Hermes Agent 环境搭建：WSL 下 GitHub 访问的三级代理方案"
date: 2026-05-05 16:00:00
tags:
  - Hermes Agent
  - WSL
  - GitHub
  - 代理
  - 网络
categories: [AI工具]
---


我的 WSL2 环境一直有个痛点：访问 GitHub 不稳定，有时能 ping 通但 curl https://github.com 超时，有时 git push 报 `Failed to connect to localhost port 7890`。今天彻底把这个问题想明白并解决了。

## 问题诊断

首先搞清楚是哪里出了问题：

```bash
# 1. ping 通 → 网络层基本OK
ping github.com

# 2. Python 能连 → TCP 443 实际通
python3 -c "import socket; s=socket.create_connection(('github.com',443),timeout=5); print('ok')"

# 3. curl 超时 → curl 被网络层拦截
curl -I --connect-timeout 5 https://github.com
```

我的情况是：**ping 能通（ICMP 110ms），Python socket 能连 443 端口，但 curl/git 的 HTTPS 请求在 TCP 层完全挂死**。这说明 WSL 宿主机网络过滤了应用层，但放行了底层 socket。

另外还发现：**用户网络封禁了 GitHub 443 端口，ghproxy.com 也被墙，Clash 关时三级全挂**。

## 三级自动切换方案

我的解法是 **~/bin/git wrapper**，每次 git 命令动态判断走哪条路：

┌──────────────────────────────────────────────────────┐
│ ~/bin/git wrapper — 三级 fallback                    │
├──────────────────────────────────────────────────────┤
│ 级1：ghproxy 通（经Clash检测） → 走 ghproxy 镜像     │
│ 级2：ghproxy 挂 + Clash 在线   → 走 Clash 代理直连   │
│ 级3：Clash 也挂了              → 直连                │
└──────────────────────────────────────────────────────┘

```bash
# 建 wrapper
mkdir -p ~/bin
cat > ~/bin/git << 'EOF'
#!/bin/bash
PROXY_URL="http://172.29.112.1:7890"
GHPROXY_TEST_URL="https://ghproxy.com"
GHPROXY_PREFIX="https://ghproxy.com"
GIT_BIN="/usr/bin/git"

# 清理上次的 rewrite，避免残留
$GIT_BIN config --global --unset url."$GHPROXY_PREFIX/https://github.com/".insteadOf 2>/dev/null

# 测 ghproxy（通过 Clash）
if curl -x "$PROXY_URL" -s --max-time 3 "$GHPROXY_TEST_URL" > /dev/null 2>&1; then
    # 级1：ghproxy 可达，走镜像
    $GIT_BIN config --global url."$GHPROXY_PREFIX/https://github.com/".insteadOf "https://github.com/"
    unset HTTPS_PROXY HTTP_PROXY http_proxy https_proxy
    exec $GIT_BIN "$@"
elif curl -x "$PROXY_URL" -s --max-time 3 https://www.google.com > /dev/null 2>&1; then
    # 级2：ghproxy 挂了但 Clash 在线，走 Clash 代理直连 GitHub
    export HTTPS_PROXY="$PROXY_URL"
    export HTTP_PROXY="$PROXY_URL"
    unset https_proxy http_proxy
    exec $GIT_BIN "$@"
else
    # 级3：Clash 也挂了，直连
    unset HTTPS_PROXY HTTP_PROXY http_proxy https_proxy
    exec $GIT_BIN "$@"
fi
EOF
chmod +x ~/bin/git

# bashrc 加 PATH 优先
sed -i '6a\
\
# 自定义 bin 目录（git wrapper）\
export PATH="$HOME/bin:$PATH"\
' ~/.bashrc
```

## 验证

```bash
source ~/.bashrc
which git  # 应该是 ~/bin/git

# 关闭 Clash 测试 tier1/tier3
# 开启 Clash 测试 tier1/tier2
git clone https://github.com/xxx/xxx.git
```

## 已知坑

1. **网络封禁 443 端口**：ping 能通 ≠ 443 端口通。企业/校园网常封 443 应用层但放行 ICMP。
2. **WSL 重启后网关 IP 可能变**：172.29.112.1 不一定仍是 WSL 网关，静默检测会失败但 wrapper 会自动 fallback。
3. **ghproxy.com 偶尔 502**：wrapper 检测到 ghproxy 不通会自动切下一级，不影响使用。
4. **Clash 关时三级全挂**：这是网络层封锁，不是代理配置问题，必须开 Clash 才能用 git。

## 总结

有了这个 wrapper 之后，日常使用完全透明：Clash 开着走代理，Clash 关了走镜像，什么都不用手动切换。
