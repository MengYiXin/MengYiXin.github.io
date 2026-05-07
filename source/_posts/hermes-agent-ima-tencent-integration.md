---
title: "Hermes Agent 接入腾讯 IMA 知识库：一次坎坷的 API 对接实录"
date: 2026-05-07 21:30:00
tags:
  - Hermes Agent
  - 腾讯IMA
  - AI知识库
  - API对接
  - 工作流
categories: [AI工具]
---

前两天把腾讯 IMA 接进了 Hermes Agent，终于可以让 AI 直接读我积累的华鲲振宇服务器配置笔记了。对接过程比预期曲折，走了不少弯路，这里把完整过程记录下来，供有类似需求的同学参考。

## 什么是腾讯 IMA

IMA 是腾讯推出的 AI 知识库产品，定位是"个人/团队的智能笔记+检索"。它的核心是**笔记管理+知识检索**，支持 API 访问，这让我们可以把它当成一个带检索能力的外部知识库来用。

对于我这种在 IMA 里积累了大量工作笔记（华鲲配置规则、行业资料）的人来说，接入 API 意味着：**以后问 Hermes 任何配置相关的问题，它可以直接去 IMA 里查，而不是靠记忆或临时搜索。**

## 我的目标

简单说就是两件事：

1. **写入**：让 Hermes 把有用的配置规则永久存进 IMA
2. **读取**：让 Hermes 根据问题从 IMA 检索相关笔记并引用

这样华鲲配置规则这类内容就能实现**长期记忆+随时调用**，不用每次都重新整理。

## 对接过程

### 第一步：安装 IMA Skill

Hermes Agent 支持通过 Skill 机制扩展功能。IMA 官方提供了一个 [ima-skill](https://github.com/nickar/ima-skill)，安装步骤很标准：

```bash
# 下载最新版本
cd /tmp
curl -sL https://github.com/nickar/ima-skill/releases/download/v1.1.7/ima-skills-1.1.7.zip -o ima.zip
unzip -q ima.zip
mv ima-skill ~/.hermes/skills/
```

安装完成后目录结构：

```
~/.hermes/skills/ima-skill/
├── SKILL.md              # Skill 定义文件
├── ima_api.cjs           # Node.js API 封装
└── references/           # 参考文档
```

### 第二步：创建 IMA 应用，获取凭证

IMA API 需要三样东西：

| 凭证 | 说明 |
|------|------|
| Client ID | 应用唯一标识 |
| API Key | 访问密钥 |
| 群 ID（可选） | 指定知识库群组 |

在 IMA 控制台创建应用后，就能拿到 Client ID 和 API Key。这里有个**坑**：IMA 的 API Key 有有效期限制，我的 Key 到 **2026-05-27** 过期，后续用之前得记得续期。

凭证存到本地配置文件方便调用：

```bash
mkdir -p ~/.config/ima
echo "67564d25e3ec5101a938e0aa22e87145" > ~/.config/ima/client_id
echo "你的API密钥" > ~/.config/ima/api_key
```

### 第三步：API 连通性验证

这是第一个卡点。

IMA Skill 自带了一个 Node.js 封装 `ima_api.cjs`，但在我环境里多次调用都**没有任何输出**，既不报错也不返回数据。原因不明，怀疑是 Node.js 版本或环境问题。

最终绕过这个封装，**直接用 curl 调用 IMA API**：

```bash
curl -s -X POST https://ima.qq.com/openapi/note/v1/list_notebook \
  -H "ima-openapi-clientid: 你的ClientID" \
  -H "ima-openapi-apikey: 你的APIKey" \
  -H "ima-openapi-ctx: skill_version=1.1.7" \
  -H "Content-Type: application/json" \
  -d '{"cursor":"0","limit":10}'
```

成功返回笔记本列表，说明凭证没问题，API 是通的。

### 第四步：搜索华鲲笔记

用关键词"华鲲"搜索 IMA，返回了 9 篇相关笔记：

| 序号 | 笔记标题 | note_id |
|------|----------|----------|
| 1 | 总规则 | 7450130605176421 |
| 2 | 配置规则 一 | 7449464528731269 |
| 3 | 配置规则 二 | 7449618136701275 |
| 4 | 配置规则 三 | 7449798412105366 |
| 5 | 配置规则 四 | 7449617432058498 |
| 6 | 配置规则 五 | 7449806943293677 |
| 7 | 配置规则 六-昇腾 | 7450126461179959 |
| 8 | 配置规则 七-昇腾 | 7450124745732506 |
| 9 | 配置规则 八 | 7457974016567714 |

### 第五步：逐篇读取笔记全文

用 `get_doc_content` 接口逐篇拉取完整内容。这个接口接受 `note_id` 和 `target_content_format` 参数：

```bash
curl -s -X POST https://ima.qq.com/openapi/note/v1/get_doc_content \
  -H "ima-openapi-clientid: 你的ClientID" \
  -H "ima-openapi-apikey: 你的APIKey" \
  -H "ima-openapi-ctx: skill_version=1.1.7" \
  -H "Content-Type: application/json" \
  -d '{"note_id":"7450130605176421","target_content_format":0}'
```

顺利拿到全部 9 篇笔记的原文，总字数超过 15000 字。

## 存入 Hermes：两种策略

拿到笔记后，接下来的问题是：**怎么让 Hermes 记住这些规则？**

我用了两种互补的策略：

### 策略一：建立独立 Skill

Hermes 的 Skill 机制支持存放任意长度的内容，没有字符数限制。把完整原文存成独立 Skill，华鲲规则就变成了 Hermes 的"内置知识"：

```
~/.hermes/skills/knowledge/huakun-config-rules/
├── SKILL.md                        # 摘要版（供快速参考）
└── references/                     # 完整原文
    ├── 01_总规则.json
    ├── 02_配置规则一.json
    ├── 03_配置规则二.json
    ├── 04_配置规则三.json
    ├── 05_配置规则四.json
    ├── 06_配置规则五.json
    ├── 07_配置规则六昇腾算力分级.json
    ├── 08_配置规则七昇腾实操细节.json
    └── 09_配置规则八.json
```

触发条件：用户问华鲲配置、服务器选型、GPU配卡、内存槽位、硬盘接口等问题时自动调用。

### 策略二：Memory 索引

在 Hermes 的持久记忆里存一条精简索引，包含：

- 核心 9 条踩坑点
- IMA 凭证信息（Key 有效期）
- API 调用示例

这样即使 Skill 没加载，问一些通用配置问题时 Hermes 也能快速答上来。

**这里遇到了一个现实问题**：Memory 上限只有 2200 字符，存了华鲲规则索引后空间剩 104 字符，非常紧张。独立 Skill 的存在大大缓解了这个压力。

## 踩坑总结

┌──────────────────────────────────────────────────────────┐
│ 本次对接踩坑汇总                                          │
├──────────────────────────────────────────────────────────┤
│ 坑1：Node.js 封装无输出   → 直接用 curl 绕过              │
│ 坑2：IMA API Key 有有效期 → 需定期续期（当前至2026-05-27）│
│ 坑3：Memory 容量紧张     → 重要内容移至独立 Skill 存储    │
│ 坑4：错误接口路径         → list_notebooks（复数）→ 实际是 │
│                             list_notebook（单数）+ search_note│
└──────────────────────────────────────────────────────────┘

## 现在的效果

接入完成后，问 Hermes 任何华鲲配置相关问题，它会：

1. 加载 `huakun-config-rules` Skill
2. 根据问题类型检索 references 里的相关 JSON
3. 结合完整原文回答，同时引用具体规则来源

对于配置选型、核查、投标报价这些场景，AI 不再靠记忆或猜测，而是有据可查。

## 写在最后

这次对接的核心收获是：**知识库的价值在于调用频率**。IMA 里的笔记如果只是存着不用，时间久了连自己都懒得翻。接进 Hermes 之后，每一次问配置问题都是一次检索和应用，知识才真正流动起来。

后续打算把更多工作笔记逐步接进来，特别是行业资料和销售话术方向。也有计划把 IMA 的写入功能用起来，让 Hermes 在对话过程中自动把有价值的内容沉淀回 IMA。

有同样想接 IMA 的同学，欢迎交流。
