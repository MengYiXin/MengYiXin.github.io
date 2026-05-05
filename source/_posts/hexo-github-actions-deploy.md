---
title: "Hexo博客的GitHub Actions自动化部署：两分支模式实战"
date: 2026-05-05 16:30:00
tags: [Hexo, GitHub Actions, 博客, DevOps]
categories: [技术实战]
---

我的博客 mengyx.com.cn 托管在 GitHub Pages，最初用 Hexo 的 `hexo deploy` 手动发布，后来踩了几个坑之后切成了 GitHub Actions 两分支模式，彻底实现了"写完文章自动上线"。这篇文章记录全过程。

## 旧架构的问题

原来的方式是 `hexo deploy` 通过 `.deploy_git/` 往 master 分支推 HTML。问题是：

1. 本地要装 Hexo 环境和部署依赖
2. 换了电脑要重新配环境
3. 图片和主题处理混乱
4. 无法做构建验证

## 新架构：两分支 + Actions

┌──────────────────────────────────────────────────────────────┐
│                      新架构                                  │
├──────────────────────────────────────────────────────────────┤
│  source 分支  ──push──►  GitHub Actions                      │
│  (Hexo源文件)              │                                  │
│                           └── hexo generate                   │
│                                    │                         │
│                           ┌──────▼──────┐                    │
│                           │  public/    │                    │
│                           └──────┬──────┘                    │
│                                    │                         │
│                           clone master 分支                   │
│                                    │                         │
│                           copy public/* + CNAME              │
│                                    │                         │
│                           push ──► master 分支               │
│                                    │                         │
│                           GitHub Pages serve master           │
└──────────────────────────────────────────────────────────────┘

- `source` 分支：存 Hexo 源文件（md、_config.yml、themes通过npm安装）
- `master` 分支：只存生成的 HTML 静态文件，由 Actions 自动维护

## 关键配置

### 1. SSH 部署密钥

Actions 往 master 分支推 HTML 需要认证。用的是 ED25519 部署密钥：

- 公钥已作为 Deploy key 添加到仓库（可写权限）
- 私钥加密存为 GitHub Secret `DEPLOY_KEY`

不要删除这个 key，否则 Actions 无法部署。

### 2. CNAME 文件

自定义域名 `mengyx.com.cn` 的 CNAME 文件：

```
# 放在 source/CNAME（注意不是 CNAME.txt）
mengyx.com.cn
```

**坑**：deploy.yml 里要显式复制这个文件：

```yaml
- name: Deploy to GitHub Pages
  run: |
    # ... hexo generate 输出到 public/ ...
    cp source/CNAME _deploy/CNAME   # 必须！Actions不会自动带
    cd _deploy
    git add .
    git commit -m "Deploy from Actions"
    git push --force origin master
```

### 3. themes/keep 是空的

Hexo 主题 `hexo-theme-keep` 是通过 npm 安装的，不在 git 仓库里：

```bash
npm install hexo-theme-keep
```

Actions 里的 `npm ci` 会自动恢复主题，所以 `themes/keep/` 目录在仓库里是空的。

### 4. 图片存放

博客图片统一存在 https://github.com/MengYiXin/blog-images，引用格式：

```
https://raw.githubusercontent.com/MengYiXin/blog-images/main/文件名.png
```

## 踩过的坑

### 坑1：中文文件名导致404

GitHub Pages 对中文路径有历史遗留 bug，访问 `https://mengyixin.github.io/2026/05/05/中文标题/` 会404。

解法：文件名必须用英文 slug（YYYY-MM-DD-slug.md 或纯英文）。

### 坑2：url配置

`_config.yml` 里 `url: http://example.com` 是默认值，构建后会导致页面跳转错误。

必须改成：`url: https://mengyx.com.cn`

### 坑3：菜单项不显示

Keep 主题的 menu 项在 Actions 构建后可能不生效。必须在主 `_config.yml` 里显式覆盖：

```yaml
theme_config:
  menu:
    Home: /
    Archives: /archives
    Tags: /tags
    About: /about
```

### 坑4：评论系统

原来用的 Valine 依赖 LeanCloud，但 LeanCloud 已关闭新用户注册且现有应用匿名功能也关了。那组 appid/appkey 是官方占位符示例，根本不能用。

建议直接换 Twikoo（腾讯云开发，免费）。

## 现在的写作流程

```
写文章（英文slug）.md  →  git add  →  git push origin source
                                        │
                         GitHub Actions 自动构建（约2-3分钟）
                                        │
                              新文章上线 mengyx.com.cn
```

全程不需要手动 hexo generate / hexo deploy，换电脑也能写。
