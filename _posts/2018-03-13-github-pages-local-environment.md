---
title: github pages本地环境配置
layout: post
comments: true
categories:
- Tutorial
tags:
- github pages
- jekyll
---

# 2019.11.06 重写

在win10下搭建github pages的本地环境真是坎坷多舛，各种错误，而且可用的文档也相对较少，中文的更是寥寥无几。。

主要问题有以下几点：

- jekyll以及其包管理、开发环境的搭建
- github pages环境的配置

<!-- more -->

要做到第一点，让所有的依赖符合要求，要么vpn，要么换国内源。


# 1. 下载安装ruby

下载 [rubyinstaller forwindows](https://rubyinstaller.org/)，注意版本号，比如我在2019.11.06操作时发现，最新的 ruby 2.6和我的Gemfile.lock中的部分组件冲突，不得已下载了2.5的旧版才成功。

安装完成后输入'ridk install'来安装MSYS2, gems等必须组件。问选项的时候一路回车即可。

# 2. 安装bundler

`gem install bundler`


# 3. 进入博客的repository目录

```shell
bundle install
bundle exec jekyll server
```


# 4.  解决“No GitHub API authentication”

- github: [metadata]

- 填写 `_config.yml` 中的站点description