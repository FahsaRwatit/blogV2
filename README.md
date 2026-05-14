# fahsa's blog

参照 Hugo Stack 主题风格的纯静态博客。

## 目录结构

```
├── index.html          # 博客前端（唯一 HTML）
├── config.json         # 站点配置、分类、密码
├── posts/
│   ├── Go/             # 每个分类一个文件夹
│   │   └── *.md
│   ├── Redis/
│   ├── Linux/
│   ├── MySQL/
│   ├── Python/
│   └── Other/
└── .nojekyll
```

## 文章格式

```markdown
---
title: 文章标题
date: 2025-12-10
tags: ["Go", "GC"]
excerpt: 一句话摘要
---

正文内容……
```

## 管理密码

默认密码：**admin**

在博客「管理员」→ 输入密码登录后可：
- 写文章（在线编辑 + 生成 Markdown）
- 修改博客设置、个人简介、分类
- 修改密码

修改密码后，将新的 `passwordHash` 同步到 `config.json` 中。

## 添加分类

1. 在 `posts/` 下新建文件夹（文件夹名 = 分类 ID）
2. 博客设置里添加对应分类（名称、颜色）
3. 文章 front-matter 的 `category` 字段填写文件夹名

## 部署

```bash
git add .
git commit -m "init"
git push
```

GitHub Pages → Settings → Pages → Deploy from branch → main / root
