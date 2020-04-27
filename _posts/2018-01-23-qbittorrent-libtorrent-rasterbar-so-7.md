---
layout: post
title: debian8编译qbittorrent提示libtorrent-rasterbar.so.7缺失的解决
categories:
  - ECM
tags:
  - linux
  - debian
  - qbittorrent
---
debian8 x64 编译 qbittorrent 3.3.11.

根据教程[Compiling-qBittorrent-on-Debian-and-Ubuntu](https://github.com/qbittorrent/qBittorrent/wiki/Compiling-qBittorrent-on-Debian-and-Ubuntu)和 [Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(15.04-and-newer)](https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(15.04-and-newer)进行操作，最终提示：

> error while loading shared libraries: libtorrent-rasterbar.so.7: cannot open shared object file: No such file or directory</blockquote>

解决方法 参考：http://blog.csdn.net/layrong/article/details/38388703

这种问题的原因有两个：

<!-- more -->

1. 一个是操作系统里确实没有包含该共享库(lib*.so.*文件)或者共享库版本不对, 遇到这种情况那就去网上下载并安装上即可.
2. 另外一个原因就是已经安装了该共享库, 但执行需要调用该共享库的程序的时候, 程序按照默认共享库路径找不到该共享库文件.  所以安装共享库后要注意共享库路径设置问题, 如下:

- 如果共享库文件安装到了/lib或/usr/lib目录下, 那么需执行一下ldconfig命令。这个命令的用途, 主要是在默认搜寻目录(/lib和/usr/lib)以及动态库配置文件/etc/ld.so.conf内所列的目录下, 搜索出可共享的动态链接库(格式如lib*.so*), 进而创建出动态装入程序(ld.so)所需的连接和缓存文件. 缓存文件默认为/etc/ld.so.cache, 此文件保存已排好序的动态链接库名字列表.

- 如果共享库文件安装到了/usr/local/lib(很多开源的共享库都会安装到该目录下)或其它"非/lib或/usr/lib"目录下, 那么在执行ldconfig命令前, 还要把新共享库目录加入到共享库配置文件/etc/ld.so.conf中, 如下:

```
# cat /etc/ld.so.conf
include ld.so.conf.d/*.conf
# echo "/usr/local/lib" >> /etc/ld.so.conf
# ldconfig
```
- 如果共享库文件安装到了其它"非/lib或/usr/lib" 目录下,  但是又不想在/etc/ld.so.conf中加路径(或者是没有权限加路径). 那可以export一个全局变量LD_LIBRARY_PATH, 然后运行程序的时候就会去这个目录中找共享库.

export LD_LIBRARY_PATH=/usr/local/mysql/lib:$LD_LIBRARY_PATH

一般来讲这只是一种临时的解决方案, 在没有权限或临时需要的时候使用.

- 如果程序需要的库文件比系统目前存在的村文件版本低，可以做一个链接
比如：
> error while loading shared libraries: libncurses.so.4: cannot open shared
> object file: No such file or directory

```
ls /usr/lib/libncu*
/usr/lib/libncurses.a   /usr/lib/libncurses.so.5
/usr/lib/libncurses.so  /usr/lib/libncurses.so.5.3
```
可见虽然没有`libncurses.so.4`，但有`libncurses.so.5`，是可以向下兼容的
建一个链接就好了

```
ln -s  /usr/lib/libncurses.so.5.3  /usr/lib/libncurses.so.4
```
我使用

```
sudo ldconfig
```
即解决。

此外，如编译安装，在/etc/systemd/system/qbittorrent.service文件中的地址因该是：/usr/local/bin
