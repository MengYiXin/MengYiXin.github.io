---
title: "博客样式丢失排查记：GitHub Pages 双构建覆盖问题"
date: 2026-05-18 13:30:00
tags: [GitHub Pages, Hexo, Troubleshooting, GitHub Actions, Cloudflare]
categories: [技术]
---

# 博客样式丢失排查记：GitHub Pages 双构建覆盖问题

## 问题现象

博客上线后，背景图没了，侧边栏不显示，评论区也没了。但文章列表、标题、菜单都正常，CSS 文件（64KB）返回 HTTP 200，图片也正常。

无痕模式也一样。

## 初步判断

按 F12 → Console 输入 `KEEP.theme_config`，发现 `first_screen.enable: false`，但我本地 `_config.yml` 明明设的是 `true`：

```yaml
theme_config:
  first_screen:
    enable: true
    background_img: /images/bg.svg
    description: 不要熬夜哟！
```

为什么会这样？

## 排查过程

### 1. 先确认服务端实际文件

```bash
# GitHub 仓库里的 master 分支
curl "https://raw.githubusercontent.com/MengYiXin/MengYiXin.github.io/master/index.html" | grep first_screen
# → enable: false ❌

# 用户访问的实际域名
curl "https://mengyx.com.cn/" | grep first_screen
# → enable: false ❌

# 本地 hexo generate 输出的文件
grep first_screen public/index.html
# → enable: true ✅
```

本地生成的是对的，但 GitHub 上是错的。

### 2. 梳理发布历史

回忆了一下，之前为了绕过 WSL SSH key 问题，执行过 `hexo deploy --force`，强制覆盖了 master 分支。而 master 分支本来是 GitHub Actions 的 `hexo generate` 输出。

`--force` 之后，本地生成的版本（`enable: true`）推上去了，但 GitHub Actions 之前生成的内容（`enable: false`）也在 master 上。两者混在一起，乱了。

### 3. 发现双重构建机制

查 GitHub Actions runs 发现一个关键问题：

```
2cf787d success 2026-05-18T05:10:57 pages build and deployment
fcbfb21e success 2026-05-18T04:48:52 pages build and deployment
07346830 success 2026-05-18T04:48:30 Deploy Hexo Blog
```

**双重构建：**

1. **Deploy Hexo Blog**（我们自己的 Actions）：`source` push → `hexo generate` → 推 master
2. **pages build and deployment**（GitHub Pages 内置）：master push → Jekyll 重新构建 → 覆盖 master

问题就在这里：我们的 Actions 刚推完正确的 `enable: true` 到 master，pages build 又跑了一遍，用 Jekyll 重新生成，把 `enable: false`（主题默认值）覆盖回去了。

### 4. 为什么 Actions 输出也不对？

还有一个问题：Actions 的 `hexo generate` 本身输出的也是 `enable: false`。

排查发现，`.github/workflows/deploy.yml` 用 sparse checkout 只拉了部分文件：

```yaml
sparse-checkout: |
  source
  themes
  _config.yml
  scaffolds
  package.json
  package-lock.json
```

但 `source/_config.yml` 这个路径在 git 里其实不存在（它就是根目录的 `_config.yml`，checkout 后直接在根目录）。真正的问题是：Hexo 在 merge `theme_config` 时，可能在某些环境下的行为不一致。

不过这不是今天的根本原因，根本原因是**双重构建覆盖**。

## 解决过程

### 第一步：本地生成正确版本

```bash
cd /mnt/f/D盘/blog 20211107
npx hexo generate
# 确认输出
grep first_screen public/index.html
# → enable: true ✅
```

### 第二步：手动推 master（绕过 Actions）

```bash
cd .deploy_git
git pull origin master
rm -rf *
cp -r ../public/* .
# 复制 CNAME
cp ../source/CNAME .
git add -A
git commit -m "deploy: fix first_screen - local generate 20260518"
git push origin master --force
```

### 第三步（关键）：关闭 GitHub Pages 自动构建

改 `build_type` 从 `workflow`（Actions）到 `legacy`（直接 serve master）：

```bash
curl -X PUT "https://api.github.com/repos/MengYiXin/MengYiXin.github.io/pages" \
  -H "Authorization: token <TOKEN>" \
  -H "Content-Type: application/json" \
  -H "Accept: application/vnd.github.v3+json" \
  -d '{"build_type": "legacy", "source": {"branch": "master", "path": "/"}}'
```

验证：
```bash
curl "https://api.github.com/repos/MengYiXin/MengYiXin.github.io/pages" \
  -H "Accept: application/vnd.github.v3+json"
# → build_type: "legacy" ✅
```

## 最终效果

```
build_type: legacy ✅
first_screen.enable: true ✅
描述: 不要熬夜哟！✅
```

背景图、侧边栏、评论区全部恢复正常。

## 经验总结

1. **永远不要 `hexo deploy --force` 推 master**。master 是 GitHub Actions 的输出，本地 generate 无法完全覆盖，force push 会丢失 Actions 生成的部分资源。

2. **GitHub Pages 存在双重构建覆盖问题**。我们的 Actions 推 master 后，GitHub Pages 的内置 build 会用 Jekyll 重新构建并覆盖内容。解决方法是改 `build_type` 为 `legacy`。

3. **发布文章的正确流程**：
   ```bash
   git push origin source   # 只推 source，Actions 自动处理剩余步骤
   ```

4. **诊断命令**：
   ```bash
   # 检查 GitHub Pages 配置
   curl "https://api.github.com/repos/<user>/<repo>/pages" \
     -H "Accept: application/vnd.github.v3+json"
   
   # 确认实际服务内容
   curl "https://<domain>/" | grep first_screen
   
   # 确认 GitHub 仓库内容
   curl "https://raw.githubusercontent.com/<user>/<repo>/master/index.html" | grep first_screen
   ```

---

*博客：mengyx.com.cn*
*日期：2026-05-18*
