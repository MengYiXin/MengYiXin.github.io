---
title: WSL 下 hexo deploy 失败排障：SSH、HTTPS Token、.deploy_git 全记录
date: 2026-05-18 12:00:00
categories:
  - 技术记录
  - DevOps
tags:
  - Hexo
  - GitHub Pages
  - WSL
  - Git
  - Deployment
---

# WSL 下 hexo deploy 失败排障：SSH、HTTPS Token、.deploy_git 全记录

## 问题背景

在 WSL 环境下执行 `hexo deploy`，`hexo generate` 成功，但推送到 GitHub Pages master 分支时报错：

```
Host key verification failed.
fatal: Could not read from remote repository.
```

hexo deploy 走的是 `.deploy_git/` 目录里的 Git 仓库，remote 配置是 SSH 格式：

```
git@github.com:MengYiXin/MengYiXin.github.io.git
```

但 WSL 环境没有配置这个仓库的 SSH key，所以 push 失败。

## 完整排查链路

### 第一层：SSH key 不存在

WSL 下没有 SSH key，git remote 又是 SSH 格式，所以直接报 `Host key verification failed`。

**尝试方案**：切到 HTTPS 格式。

### 第二层：HTTPS Token 嵌入 URL 失败

修改 remote 为带 token 的 HTTPS URL：

```bash
git remote set-url origin https://ghp_TOKEN@github.com/MengYiXin/MengYiXin.github.io.git
git push origin master
```

仍然失败，因为 ~/bin/git wrapper 会把 URL rewrite 到 `ghproxy.com`，但 ghproxy 不认识带 token 的 URL 格式。

### 第三层：credential helper 干扰

尝试用 credential helper 存储 token：

```bash
git config --global credential.helper "store"
git push origin master
```

失败：`could not read Username for 'https://github.com': terminal prompts disabled`

credential helper 没有生效，git 在尝试读取 stdin。

### 第四层：直接绕过 wrapper

发现问题根源：~/bin/git wrapper 在检测到代理可用时，会把 GitHub URL rewrite 到 ghproxy.com。但 ghproxy 不认识带 token 的 HTTPS URL，导致连接失败。

解决：清理所有 git config 的 rewrite 规则，直接推。

```bash
# 清理 git wrapper 的 rewrite
git config --global --unset url."https://ghproxy.com/https://github.com/".insteadOf

# 用 system git 直接推
/usr/bin/git push origin master
```

仍然失败：报错 `rejected because the remote contains work that you do not have locally`。

### 第五层：.deploy_git 的 remote 问题

进入 `.deploy_git/` 目录检查 remote 配置：

```bash
cd .deploy_git
git remote -v
# 输出：git@github.com:MengYiXin/MengYiXin.github.io.git (push)
```

原来 `.deploy_git/` 是独立的 Git 仓库，remote 指向 SSH URL，且没有 fetch。

### 第六层：修复 .deploy_git 的 remote

给 `.deploy_git/` 添加 HTTPS remote：

```bash
git remote add origin https://ghp_TOKEN@github.com/MengYiXin/MengYiXin.github.io.git
git remote -v
# 确认 remote 已改
```

然后 push，但因为远程有 GitHub Actions 生成的提交，本地没有，直接 push 会失败。

### 第七层：--force 覆盖

GitHub Pages 的 master 分支是 GitHub Actions 自动生成的，本地 hexo generate 的结果不需要保留远程历史，直接覆盖：

```bash
GIT_TERMINAL_PROMPT=0 git push origin master --force
```

成功！

```
To https://github.com/MengYiXin/MengYiXin.github.io.git
 + 386aa93...e211b24 master -> master (forced update)
```

## 为什么 GitHub Actions 部署没问题，本地 hexo deploy 就挂了？

GitHub Actions 用的是**部署密钥（Deploy Key）**，是 SSH 格式，但 Actions runner 有这个 key，所以能推成功。

WSL 本地没有这个 SSH key，所以推不上去。

## 教训总结

| 层级 | 问题 | 解法 |
|------|------|------|
| 1 | SSH key 缺失 | 切 HTTPS |
| 2 | ~/bin/git wrapper rewrite | 用 /usr/bin/git |
| 3 | credential helper 不生效 | token 直接嵌入 URL |
| 4 | .deploy_git 是独立仓库 | 单独修复它的 remote |
| 5 | 远程有提交冲突 | --force 覆盖 |

## 预防方案

下次 hexo deploy 前，先检查 `.deploy_git/` 的 remote 配置：

```bash
cat .deploy_git/.git/config | grep url
# 应该是 HTTPS 格式，不是 SSH
```

如果看到 `git@github.com:` 开头，手动改成 HTTPS：

```bash
cd .deploy_git
git remote set-url origin https://github.com/MengYiXin/MengYiXin.github.io.git
```

## 关键路径

```
.hexo-deploy/          # Hexo 自动生成的临时部署目录
  .deploy_git/        # 实际执行 git push 的仓库
    .git/config       # 关键：remote 配置在这里
```

Hexo generate 的输出先到 `.deploy_git/` 再 push，所以 `.deploy_git/` 的 Git 配置是最后一道关卡。