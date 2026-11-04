# saya-posts

`saya.work` 的**文章源仓库**。站点正文不落数据库 —— 数据库只存元数据，正文由读者浏览器直接从这里拉取。

站点：<https://saya.work>

## 目录

| 目录 | 内容 | 命名 |
| --- | --- | --- |
| `news/` | AI 资讯（每日早间） | `news-digest-YYYY-MM-DD.md` |
| `job/` | 求职资讯（每日午间） | `job-digest-YYYY-MM-DD.md` |
| `kb/` | 知识科普（每日晚间） | `kb-YYYY-MM-DD.md` |
| `essays/` | 随笔与创刊号（人工撰写） | 任意 |

`docs/automation/` 存放自动化的采集规范与文章模板副本。

## 文章结构

每篇文件以 front-matter 开头，之后是 Markdown 正文：

```yaml
---
title: 千问接入 Apple 智能今日开测，Gemini 4 Carbon 登顶
date: 2026-11-05T07:00:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 150-250 字的完整摘要……
---
```

## 怎么读

**在站点读**（推荐）：<https://saya.work/post/news-digest-2026-11-05>

**命令行**：

```bash
curl -L https://raw.githubusercontent.com/saya-ch/saya-posts/main/news/news-digest-2026-11-05.md
```

**本地克隆**：

```bash
git clone https://github.com/saya-ch/saya-posts.git
```

## 元数据接口

文章列表（含标题、摘要、标签、发布时间、原文 URL，不含正文）：

```
GET https://saya.work/api/public/posts
```

## 说明

- `news/` 与 `job/` 由自动化流水线生成，事实与数据以每条给出的原文链接为准；发现错误可提 issue。
- `kb/` 为技术科普，`essays/` 为人工撰写，观点仅代表作者。
- 许可条款见 [LICENSE](LICENSE)。
