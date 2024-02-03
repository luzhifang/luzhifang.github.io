# 基于Hugo + Github Pages搭建LoveIt主题个人博客


## 1. 注册 [Github](https://github.com/) 账号并创建仓库

- 注册账号和创建仓库此不赘述，说一下需要注意的点

1. 仓库的名字必须以 github.io 结尾
2. Repository name 需要与 Owner 一致，不一致的话会多一个层级。例：owner.github.io
3. 仓库可见性必须是 public

## 2. 安装 hugo

1. windows
   1. [下载安装包](https://github.com/gohugoio/hugo/releases/download/v0.104.3/hugo_extended_0.104.3_windows-amd64.zip)
   2. 解压并发执行文件所在目录添加到环境变量
2. linux
   1. [下载安装包](https://github.com/gohugoio/hugo/releases/download/v0.104.3/hugo_0.104.3_Linux-64bit.tar.gz)
   2. 解压并发执行文件所在目录添加到环境变量
   3. /etc/profile 文件添加一行(export 解压目录)，执行命令 source /etc/profile 使配置文件生效

## 3. 安装[git](https://git-scm.com/downloads)

## 4. 创建站点和配置主题

1. 选定一个站点放置目录执行一下命令

```Shell
hugo new site myblog
cd myblog
git init
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
cp myblog/themes/LoveIt/exampleSite/config.toml .
hugo serve
```

2. 浏览器打开 http://localhost:1313/
3. 修改 config.toml 以满足自己需要，里面有详细注释

## 5. 发布内容

1. 执行 hugo new posts/first_post.md
2. 添加头部内容

```Markdown
---
title: 标题
author: 作者
date: 日期(格式：yyyy-MM-dd)

// 添加分类
categories:
  - Hugo

// 添加标签
tags:
  - Hugo
  - Github Pages
  - LoveIt
---
```

3. 接下来就是编写的 markdown 文档了

## 6. 将博客内容上传至 Github

1. 在 myblog 目录下执行一下命令

```Shell
hugo
cd public
git init
git add .
git commit -m "first post"
git remote add origin https://github.com/yourusername/yourusername.github.io
git push -u origin main
```

2. 进入你的 yourusername.github.io 仓库看文件是否上传成功
3. 访问 `https://yourusername.github.io`

