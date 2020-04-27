---
layout: post
title: WR-703N 安装openwrt到外置储存
categories:
  - Tutorial
tags:
  - openwrt
  - 703n
---
这两天折腾了下openwrt，起因是发现了一个深藏的703n，这个古老的小东西竟然非常受openwrt社区欢迎，一直是主要支持的设备之一。

根据https://wiki.openwrt.org/toh/tp-link/tl-wr703n，

安装了openwrt其中也是颇为曲折，在windows下的curl不好找，最新版的都是带ssl的，使用sftp各种错误，后来在

http://www.right.com.cn/forum/forum.php?mod=viewthread&amp;tid=159078&amp;highlight=703n

的2楼找到了先贤做好的各种文件，直接就可以刷了，结合这两篇文章总算是刷好了openwrt，版本新的吓人，DESIGNATED DRIVER (Bleeding Edge, r49379) ，空间也是占用的很多，想玩儿必须是装个外接u盘了，很可惜网上这方面的信息比较少，大多数也很旧，折腾了半天，发现都成功不了。以前玩儿vps，都没有搞过挂载这回事儿啊，无奈跑去openwrt的wiki发现，人家早就更新了最新的信息啊。。。参见：

https://wiki.openwrt.org/doc/howto/extroot

<!-- more -->

这里想说的是，不要跟着网上的其他教程贴子瞎搞，都太老了。。。另外，703n的空间实在是太小了，大约400+kb，

```shell
opkg install block-mount kmod-fs-ext4 kmod-usb-storage-extras</pre>
```
就足以占个七七八八了，其他的都先别装，跟着教程走完自然就海阔天空啦！

主要有两条命令：
1. Prepare your external storage root overlay _copy files from existing /overlay into new root_

```
mount /dev/sda1 /mnt ; tar -C /overlay -cvf - . | tar -C /mnt -xf - ; umount /mnt
```
2. Create fstab with the following command

```
block detect > /etc/config/fstab; \
   sed -i s/option$'\t'enabled$'\t'\'0\'/option$'\t'enabled$'\t'\'1\'/ /etc/config/fstab; \
   sed -i s#/mnt/sda1#/overlay# /etc/config/fstab; \
   cat /etc/config/fstab;
```
还有windows下将u盘分区并格式化一部分为ext4和swap的工具可以用minitool，很好用，自己搜一下哈。

就酱


其他参考：https://www.coldawn.com/tl-wr703n-tl-wr720n-official-openwrt-barrier_breaker-14-07-extroot-rootfs-on-remove-disk/

试了一下Chaos Calmer 15.05.1，安装完成后剩余空间只有280kb，最经典的是删除固件自带的程序后这个空间会进一步缩小，是没有办法实现本文的文件转移的，因为kmod-fs-ext4就需要200+空间的，无奈再好换回DESIGNATED DRIVER。想了想，可能是openwrt固件里编译好的程序没有办法删除吧，删除了其实相当于在rom中写入了些什么文件用来屏蔽想删除的程序，因而越删越小？猜测。。

不能挂载fat32的情况，

```
dmesg | tail
```
可见缺少kmod-nls-cp437     kmod-nls-iso8859-1     kmod-nls-utf8，安装后

```
mount -t vfat /mnt/sda1 /data 
```
即可

参考：http://www.right.com.cn/forum/forum.php?mod=redirect&amp;goto=findpost&amp;ptid=54211&amp;pid=370118
