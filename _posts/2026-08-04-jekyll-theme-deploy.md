---
title: Jekyll主题网站部署
date: 2026-08-04 12:00:00 +0800
categories: [教程]
tags: [Jekyll, GitHub Pages, Chirpy, 部署]
---

本文记录使用 Jekyll Chirpy 主题搭建个人博客并部署到 GitHub Pages 的完整过程，以及遇到的各种问题和解决方案。

## 第一步：Fork 仓库

将 Chirpy 主题仓库 [cotes2020/jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) Fork 到自己的 GitHub 账号下，仓库名命名为 `username.github.io`（其中 `username` 替换为你自己的 GitHub 用户名）。

## 第二步：修改 `_config.yml` 配置

Clone 到本地后，修改 `_config.yml` 文件，主要修改以下内容：

| 配置项 | 修改前 | 修改后 |
|--------|--------|--------|
| `lang` | `en` | `zh-CN` |
| `title` | `Chirpy` | `夏厦的个人博客` |
| `tagline` | `A text-focused Jekyll theme` | （清空） |
| `description` | Chirpy 默认描述 | `夏厦的个人博客，记录技术学习与生活点滴。` |
| `url` | `""` | `"https://xiaxia030226.github.io"` |
| `github.username` | `github_username` | `xiaxia030226` |
| `twitter.username` | `twitter_username` | （清空） |
| `social.name` | `your_full_name` | `夏厦` |
| `social.links` | Twitter + GitHub 默认链接 | 只保留 `https://github.com/xiaxia030226` |
| `cdn` | `"https://chirpy-img.netlify.app"` | `""` |
| `avatar` | `"/commons/avatar.jpg"` | `"/assets/img/avatars/avatar.jpg"` |
| `comments.provider` | （空） | 保持为空，不使用评论系统 |

## 第三步：配置 GitHub Actions 部署

进入 GitHub 仓库的 **Settings → Pages**，在 **Source** 下拉菜单中选择 **GitHub Actions**，点击生成配置文件并 **Commit Changes**。

## 第四步：解决 Sass 找不到 Bootstrap 样式表的问题

推送后 GitHub Actions 构建失败，报错：

```
Error: Can't find stylesheet to import.
  ╷
1 │ @use 'vendors/bootstrap';
  │ ^^^^^^^^^^^^^^^^^^^^^^^^
  ╵
```

**原因分析**：Chirpy 主题使用 Bootstrap 的 SCSS 源文件，但 `_sass/vendors/` 目录下没有 Bootstrap 的文件。此外，GitHub Actions 部署工作流文件 `pages-deploy.yml` 被放在了 `.github/workflows/starter/` 子目录中，GitHub 不会识别子目录中的工作流。

**解决方案**：

1. 将 `pages-deploy.yml` 从 `.github/workflows/starter/` 移动到 `.github/workflows/` 根目录，并删除空的 `starter` 目录。

2. 在工作流中添加 Node.js 环境、`npm install` 安装依赖，然后将 Bootstrap SCSS 文件复制到 `_sass/vendors/` 目录：

```yaml
- name: Setup Node
  uses: actions/setup-node@v6
  with:
    node-version: lts/*

- name: Install npm dependencies
  run: npm install

- name: Copy Bootstrap SCSS
  run: |
    mkdir -p _sass/vendors
    cp -r node_modules/bootstrap/scss/* _sass/vendors/
```

## 第五步：解决 Bootstrap SCSS 子目录引用问题

上一步修改后再次构建，SCSS 编译虽然不再报错，但又出现了新问题——Bootstrap 的 `bootstrap.scss` 内部通过 `@import` 引用了 `mixins/`、`forms/`、`helpers/` 等子目录下的文件，而第一次只复制了根目录的 `.scss` 文件，子目录没有复制。

**解决方案**：使用 `cp -r` 递归复制整个 Bootstrap SCSS 目录结构，确保子目录也被完整复制：

```yaml
- name: Copy Bootstrap SCSS
  run: |
    mkdir -p _sass/vendors
    cp -r node_modules/bootstrap/scss/* _sass/vendors/
```

同时还需要：
- 添加 `npm run build` 步骤构建 JS 文件到 `assets/js/dist/`
- 删除 `_posts/` 下的示例文章（它们引用了不存在的图片）
- 在 htmlproofer 测试中添加 `--allow-missing-href` 参数

完整的工作流文件 `pages-deploy.yml` 关键部分如下：

```yaml
- name: Setup Node
  uses: actions/setup-node@v6
  with:
    node-version: lts/*

- name: Install npm dependencies
  run: npm install

- name: Copy Bootstrap SCSS
  run: |
    mkdir -p _sass/vendors
    cp -r node_modules/bootstrap/scss/* _sass/vendors/

- name: Build JS assets
  run: npm run build

- name: Build site
  run: bundle exec jekyll b -d "_site${{ steps.pages.outputs.base_path }}"
  env:
    JEKYLL_ENV: "production"
```

## 总结

经过以上步骤，博客成功部署上线。回顾整个过程，关键点在于：

1. GitHub Actions 工作流文件必须放在 `.github/workflows/` 根目录
2. Chirpy 主题依赖 Bootstrap 的 SCSS 源文件，需要在 CI 中通过 npm 安装并复制
3. 使用 `cp -r` 递归复制确保子目录文件齐全
4. JS 文件也需要通过 `npm run build` 构建
5. 删除示例文章避免图片引用错误

部署成功后访问 `https://你的用户名.github.io` 即可看到博客。
