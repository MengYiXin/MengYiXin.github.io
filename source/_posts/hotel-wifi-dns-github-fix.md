---
title: "住酒店顺手修好的GitHub访问问题：DNS污染"
date: 2026-05-06 01:00:00
tags:
  - GitHub
  - 网络
  - DNS
  - 代理
  - WSL2
  - 故障排除
categories: [技术实战]
---

## 从一个奇怪的WiFi故障说起

昨晚住桔子水晶酒店，笔记本连上WiFi后华住会员 portal 死活跳不出来，页面转圈然后报超时。同一个WiFi，手机能跳、电脑能跳，唯独我的拯救者Y7000P不行。

酒店前台说"改成114.114 DNS就好了"，我试了果然立刻通了。然后顺手测了一下 GitHub——居然也通了。

这就奇怪了：我没有开任何代理，Clash 关着，git proxy heal 也没触发，怎么突然 GitHub 就通了？

## 问题根因

![配图](https://raw.githubusercontent.com/MengYiXin/blog-images/main/08-hotel-wifi-dns-fix.svg)

酒店WiFi自带的 DNS（31.1.x.x 或者 1.31.x.x，记不太清具体哪个）解析 GitHub 时返回了错误的出口IP，导致 HTTPS 443 端口连接失败或超时。

这跟"开了代理但代理挂了"完全不同——**根本不需要代理**，是 DNS 层面就出了问题。ping github.com 能通（ICMP 不走 443），但 curl https://github.com 超时，git push 挂死，现象看起来跟代理故障几乎一样。

## 怎么判断是不是 DNS 问题

网络故障时，先确认是 DNS 污染还是代理问题：

```bash
# ping 通但 curl 超时 → 很可能是 DNS 污染
ping github.com

# 直连测试 443 端口（Python 走的是系统 socket，不过代理）
python3 -c "import socket; s=socket.create_connection(('github.com',443),timeout=5); print('ok')"

# 看 DNS 解析到了什么 IP
nslookup github.com

# 对比不同 DNS 的解析结果
nslookup github.com 114.114.114.114
nslookup github.com 8.8.8.8
nslookup github.com 31.1.x.x  # 酒店 DNS
```

如果 114.114 解析出来的 IP 和酒店 DNS 解析出来的不同，那就是酒店 DNS 有污染。

## 解决方法

Windows 改 DNS（适配所有应用，包括 WSL）：

1. 控制面板 → 网络和共享中心 → WLAN属性 → IPv4
2. 首选 DNS 填 `114.114.114.114`（或 `8.8.8.8`）
3. 确定保存

![Windows DNS设置示意](https://raw.githubusercontent.com/MengYiXin/blog-images/main/08-hotel-wifi-dns-windows-setting.png)

改完立刻生效，不需要重启WiFi，也不需要开代理，GitHub 访问就正常了。

## 经验总结

┌──────────────────────────────────────────────────────┐
│ 以后遇到 GitHub 访问不稳定，先查 DNS                 │
├──────────────────────────────────────────────────────┤
│ 1. ping 通但 curl 超时  → 很可能是 DNS 污染        │
│ 2. 改 114.114.114.114  → 多数情况可解              │
│ 3. 代理配置是事后补救  → 优先查 DNS才是根本        │
└──────────────────────────────────────────────────────┘

**DNS 污染是、公共WiFi的常见问题，不只是酒店**。企业网络、校园网、机场WiFi都可能遇到类似情况——portal 跳转失败、某些 HTTPS 站点打不开，但又不是完全断网。

记住这个优先级：先查 DNS，不行再看代理配置。
