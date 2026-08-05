---
name: blog-post
description: >
  This skill should be used when adding a new article to the user's Jekyll blog (xxHtml project).
  It automatically determines the next article number for a given category, reads code files
  from the Unity project (xx) for analysis, and creates a properly formatted Markdown post.
  Trigger when the user wants to write a dev diary, add a blog post, document code modules,
  or create new articles under any blog category.
---

# Blog Post Creation Skill

## Purpose

Create new blog posts in the user's Jekyll + Chirpy blog located at:
`c:\project\newStart\xxHtml\xiaxia030226.github.io\_posts\`

The blog sits alongside a Unity game project at `c:\project\newStart\xx\`, and most
articles analyze code modules from that Unity project.

## Blog Configuration

| Item | Value |
|------|-------|
| Framework | Jekyll + Chirpy v7.6.0 |
| Posts directory | `c:\project\newStart\xxHtml\xiaxia030226.github.io\_posts\` |
| File naming | `YYYY-MM-DD-slug.md` |
| Permalink | `/posts/:title/` |
| Categories | Defined dynamically in front matter |
| Language | `zh-CN` |
| Timezone | `Asia/Shanghai` (+0800) |
| Git remote | `https://github.com/xiaxia030226/xiaxia030226.github.io` |

## Article Front Matter Format

Every post must use this front matter:
```yaml
---
title: 编号.标题
date: YYYY-MM-DD 00:00:00 +0800
categories: [分类名]
tags: [tag1, tag2, ...]
---
```

The date should always be the current date (today).

## Article Title & Numbering

The `title` field format is `编号.标题` (e.g., `1.基础框架-单例模板`, `2.C#对象池`).

**To determine the next number for a category:**
1. Search all `*.md` files in `_posts/` directory
2. Parse each file's front matter `categories` field
3. Filter posts belonging to the target category
4. Extract the number prefix from each post's `title` field
5. The next number = max existing number + 1

## Content Generation Workflow

When the user provides:
- **Category** (e.g., "个人项目开发日记")
- **Title** (e.g., "C#对象池", without the number prefix)
- **Code folder path** (the folder whose code should be analyzed)

Follow this workflow:

### Step 1: Determine the next number

Read all existing `_posts/*.md` files. For each post whose `categories` front matter
matches the target category, extract the leading number from `title`. The new article
number = (max found) + 1. If no existing posts in the category, start at 1.

### Step 2: Read the code files

Read every `.cs` file in the specified Unity project folder (ignoring `.meta` files).
The Unity project root is `C:\project\newStart\xx\`. Understand each file's purpose
and structure.

### Step 3: Generate the article content

For each code file, create a section with:
- A heading with the file name and a brief description
- A code block showing the full source
- Key points explaining the design decisions, patterns used, and important details

If the user mentions specific problems or insights (e.g., "generics constraint issue",
"Stack vs List"), include a dedicated "编写中遇到的问题" section after the file walkthrough.

Content guidelines:
- Write in Chinese (zh-CN)
- Keep paragraphs concise
- Use tables for comparisons when helpful
- Do NOT add "使用示例" (usage example) sections unless the user explicitly asks
- Do NOT add "后续计划" (future plans) sections unless the user explicitly asks
- No emojis

### Step 4: Write the file

Create the Markdown file at the path:
```
c:\project\newStart\xxHtml\xiaxia030226.github.io\_posts\YYYY-MM-DD-{slug}.md
```
The slug should be `dev-diary-{N}` where N is the article number.

Generate appropriate tags based on the code content (e.g., Unity, C#, design patterns, etc.).

### Step 5: Confirm and offer to push

After writing the file, summarize what was created and ask the user if they want to
push. Only push if the user explicitly confirms.

## Interaction Pattern

The user will provide information in a conversational way, such as:
- "新增一篇开发日记，标题叫EventCenter，分析Singleton文件夹"
- "在个人项目开发日记下加一篇文章，讲Pool模块"

If any required information is missing (category, title, or folder path), ask the user
for clarification before proceeding.
