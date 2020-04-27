---
layout: post
title: WordPress提示 "在裁剪您的图像时发生了错误"
categories:
  - ECM
tags:
  - wordpress
  - error
---

前几天上传图片想用作site icon，但是提示:"在裁剪您的图像时发生了错误。"

英文显示，"There has been an error cropping your image."

这是因为缺少PHP GD库的原因，貌似直接装php7.0-fpm不会装这个库。没有这个库，则图片水印、缩略图、验证码等都无法使用。

解决:

<!-- more -->

Debian/Ubuntu下运行：

```shell
sudo apt-get install php7.0-gd
```
#我用的php7.0，如果是其他版本，请自行修改

CentOS下运行：

```shell
yum install php7.0-gd
```
安装PHP-GD库后

```shell
sudo service nginx restart
```
