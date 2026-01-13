# 一个静态博客 Github pages 
这是一个静态博客的网站
> 注释 ：这个Action作者是 Hux，这里是fork过来，修改了workflow部署step,并修改了一些内容和主题，用于自己使用。这个code是我用于学习Action自动部署用的。
> 当然也可以使用项目站点和自定义域名。下面是静态网站的使用说明
# 说明和使用方法
## 1._config.yml
修改主体信息
## _includes 
修改主题样式
## _posts 
```bash
# 这里存放你的文章Markdown文件。这里是Markdown的头部配置信息。与vitepress差不多。
---
layout:       post
title:        "《JavaScript 二十年》推荐语"
author:       "Hux"
header-style: text
catalog:      true
tags:
    - Web
    - JavaScript
---
主要是title和标签tags，下面就是正文了。
文件名写上日期，结尾是markdown即可。比如 2026-01-13-GithubAction使用技巧.markdown,提交完事，这里的actions就开始自动部署。
```
# workflow
```yml
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

# Sample workflow for building and deploying a Jekyll site to GitHub Pages
name: Deploy Jekyll site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["master"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@4a9ddd6f338a97768b8006bf671dfbad383215f4
        with:
          ruby-version: '3.1' # Not needed with a .ruby-version file
          # 禁用自动缓存，改用手动缓存
          bundler-cache: false
      
      # 手动配置缓存，增加容错机制
      - name: Restore gem cache
        id: cache-gems
        uses: actions/cache@v4  # 使用最新版缓存 Action
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
          restore-keys: |
            ${{ runner.os }}-gems-
          fail-on-cache-miss: false  # 缓存失败不终止流程
      
      # 安装依赖（缓存命中则跳过，未命中/缓存失败则执行）
      - name: Install dependencies
        run: |
          bundle config path vendor/bundle
          bundle install --jobs 4 --retry 3
      
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      
      - name: Build with Jekyll
        # Outputs to the './_site' directory by default
        run: bundle exec jekyll build --baseurl "${{ steps.pages.outputs.base_path }}"
        env:
          JEKYLL_ENV: production
      
      - name: Upload artifact
        # Automatically uploads an artifact from the './_site' directory by default
        uses: actions/upload-pages-artifact@v3

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
