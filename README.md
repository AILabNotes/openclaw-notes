# OpenClauw 笔记 - 小龙虾养成记录

> 按 Markdown 形式记录 OpenClaw 的使用心得

## 项目说明

这是一个基于 Jekyll 的技术博客，用于记录 OpenClaw 的日常使用心得：

- Agent 创建和配置
- Skills 开发和使用
- Agent Swarm 的实践
- 自动化脚本分享
- 问题解决经验

## 本地开发

### 安装 Jekyll

```bash
# Ubuntu/Debian
sudo apt-get install ruby-full build-essential zlib1g-dev

# 安装 Bundler
gem install bundler
```

### 启动本地服务器

```bash
cd openclaw-notes
bundle install
bundle exec jekyll serve
```

访问 http://localhost:4000

## 写作规范

### 文章命名

格式：`YYYY-MM-DD-标题.md`

例如：`2026-03-05-添加女娲agent.md`

### 文章模板

```markdown
---
layout: post
title: "文章标题"
date: 2026-03-05
categories: openclaw
---

# 标题

正文内容...
```

### 推荐分类

- `agent` - Agent 相关
- `skill` - Skills 开发和使用
- `swarm` - Agent Swarm
- `automation` - 自动化脚本
- `troubleshooting` - 问题解决
- `tips` - 技巧和心得

## GitHub Pages 部署

### 推送到 GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```

### 启用 GitHub Pages

1. 在 GitHub 仓库中，进入 Settings → Pages
2. Source 选择 `GitHub Actions`
3. 使用以下 Workflow 配置（自动创建）

---

**创建日期：** 2026-03-05
**技术栈：** Jekyll + GitHub Pages
