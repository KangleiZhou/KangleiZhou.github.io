---
title: fdisk与df显示磁盘大小不一致的解决办法
layout: post
date: 2019-11-06 12:47:00
comments: true	
categories:
- Tutorial
tags:
- linux
- ubuntu
---

昨天重装了一个vps，用了ubuntu18.04，踩了一下午的netplan坑，又发现，磁盘空间货不对板。google一下找到了解决方案。

<!-- more -->

很简单，这个问题一般是因为文件系统和分区大小不一致导致的。

```shell
resize2fs /dev/sda1
```

即可解决问题。

`resize2fs`使文件系统`filesystem`与`partition`保持一致从而提升空间。



ref: [Why are partition size and df output different?](https://unix.stackexchange.com/questions/10776/why-are-partition-size-and-df-output-different)

