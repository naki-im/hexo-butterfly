# naki-im Blog
>[!caution]
>本项目为已完成配置的Hexo项目，且包含最新文章，如果你需要自用，请确保将原始配置全部修改到位，需要一定动手能力。

一个Hexo Butterfly主题的成品项目，开箱即用（对吧？）

## 项目特性

 - 已配置umami统计
 - 支持Markdown
 - SEO优化
 - 自带minify插件，缩小构建输出体积
 - 支持RSS订阅
 - 支持多种评论系统
 - 自带文章阅读统计

## 技术栈

 - 框架：Hexo
 - 主题：Butterfly by @jerryc127
 - 包管理器：npm

## 快速上手

### 环境要求

 - Nodejs 18+
 - npm

### 安装依赖包

```bash
npm install
```

### 调试

```bash
npm run dev
```

### 构建生产版本

```bash
npm run build
```

### 预览构建结果

```bash
npm run server
```

## 使用方法

### 创建新文章

使用Hexo-cli创建

```bash
hexo n "<这里填写文章标题>"
```

### 清理构建产物

```bash
hexo clean
```

### 配置项目

修改`_config.yml`和`_config.butterfly.yml`

### 文章格式

文章使用Markdown格式，需使用Front Matter
格式：
```markdown
---
title: <文章标题>
categories:
  - <文章分类>
tags:
  - <文章标签>
  - <一行一个>
abbrlink: <构建时自动生成> 
date: <文章发布时间，格式：YYYY-MM-DD HH:mm:ss>
---

此处是文章正文
```

## 目录结构

```text
.
├── _config.butterfly.yml
├── _config.yml
├── package.json
├── package-lock.json
├── readme.md
├── scaffolds
│   ├── draft.md
│   ├── page.md
│   └── post.md
└── source
    ├── 20250824124223903.webp
    ├── about
    │   └── index.md
    ├── categories
    │   └── index.md
    ├── _data
    │   ├── link.yml
    │   └── widget.yml
    ├── friendlink
    │   └── index.md
    ├── _index.md
    ├── _posts
    │   ├── 对一些CDN和静态托管的评测.md
    │   ├── 给博客加一个线路切换.md
    │   ├── 互联网广告捞金指南.md
    │   ├── 记录.md
    │   ├── 你的网站是怎么被墙的.md
    │   ├── 如何安全地加密数据.md
    │   ├── 如何获取一个免费的ip6-arpa域名.md
    │   ├── 如何科学地消除敏感词.md
    │   ├── 使用acme-sh申请Let-s-Encrypt的TLS证书.md
    │   ├── 提升Netlify的中国大陆访问速度.md
    │   ├── 网络通信是怎么加密的.md
    │   └── 制作一个随机图API.md
    ├── privacy-policy
    │   └── index.md
    ├── robots.txt
    ├── static
    │   └── route.html
    └── tags
        └── index.md

11 directories, 31 files
```

## 部署

### 1.通过项目自带的部署

```bash
npm run deploy
```

### 2.手动部署

运行构建后，输出的文件位于`public/`目录下，手动复制到目标服务器即可。

## 贡献

欢迎提交Issues和Pull Requests

## 鸣谢

[Butterfly主题](https://github.com/jerryc127/hexo-theme-butterfly)

[Hexo](https://hexo.io)

以及所有贡献者们，此处未能全部列出

## 许可证

AGPL v3
