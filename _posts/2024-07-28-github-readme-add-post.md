---
layout: post 
title: github个人首页自动展示你最近写的blog文章
date: 2024-07-28 
tags: github
---

## 起因

看到篇文章，可以将自己的Blog最近的几篇文章自动同步在github自己的首页上。如下面：
![案例1](https://user-images.githubusercontent.com/8397274/88047382-29b8b280-cb6f-11ea-9efb-2af2b10f3e0c.png)
感觉挺有意思就弄起来了。

## 动手

原理是开发者基于 GitHub Actions 开发新的小功能。我这里用到一个开源项目叫 [blog-post-workflow](https://github.com/gautamkrishnar/blog-post-workflow)，它可以通过
RSS（订阅源）来获取到博客的最新文章。以下是实现的步骤：

1.将以下部分添加到您的 README.md 文件中，您可以指定您想要的任何标题。只需确保在自述文件中使用<!-- BLOG-POST-LIST:START --><!-- BLOG-POST-LIST:END --> 即可。工作流程将用实际的
博客文章列表替换此评论：

```md
# Blog posts (可修改这部分)
<!-- BLOG-POST-LIST:START -->
<!-- BLOG-POST-LIST:END -->
```

2、打开[https://github.com/YOUR_NAME/YOUR_NAME/actions](https://github.com/yourname/yourname/actions)，建个文件 new workflow。

3、新建一个blog-post-workflow.yml文件，复制下面的内容。
```angular2html
name: Latest blog post workflow
on:
  schedule: # Run workflow automatically
    - cron: '0 * * * *' # Runs every hour, on the hour
  workflow_dispatch: # Run workflow manually (without waiting for the cron to be called), through the GitHub Actions Workflow page directly
permissions:
  contents: write # To write the generated contents to the readme

jobs:
  update-readme-with-blog:
    name: Update this repo's README with latest blog posts
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Pull in dev.to posts
        uses: gautamkrishnar/blog-post-workflow@v1
        with:
          feed_list: "https://dev.to/feed/gautamkrishnar,https://www.gautamkrishnar.com/feed/"
```

4、修改上面的feed_list的url为你想要的Blog的RSS链接，我的Blog是jekyll搭建的，有RSS的功能，所以填了http://chunpat.com/feed.xml。

5、查看 GitHub Actions。

![github-page](img/2024-07-28-github-readme-add-post/github-page.jpg)

6、效果

![github-page](img/2024-07-28-github-readme-add-post/myhome.jpg)


## 迭代

* 2024年07月28日 14:50:54 初稿

## 参考

- 1、[GitHub Actions 还能这么玩？自动将发布的博客文章更新到 GitHub 个人主页](https://shenxianpeng.github.io/2021/11/special-repository/)
- 2、[https://github.com/gautamkrishnar/blog-post-workflow](https://github.com/gautamkrishnar/blog-post-workflow)
