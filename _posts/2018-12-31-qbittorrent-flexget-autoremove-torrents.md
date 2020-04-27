---
layout: post
title: qbittorrent、flexget和AMT配合使用，全自动的盒子配置
categories:
  - Tutorial
tags:
  - qbittorrent
  - flexget
  - autoremove-torrents
---

qbittorrent应该才是Linux下PT的正确打开方式，尽管网络上rutorrent备受推崇，transmission
因为预装在许多nas上而成为许多人唯一知道的选择，但只要有机会选择，那些好用的东西，总会
脱颖而出的。qb下载上传如此之快，甩其他客户端（不包括Deluge，没用过）不知几条街，几乎
能连上所有的种子，但也会带来一些副作用，比如。。。cpu占用过高，自行取舍哈。

qbittorrent的另一个致命问题是，其webui不支持rss，可想而知对于盒子用户而言这是多么不可理喻
的事情，但为了qb的高效率，还是忍了。。。由于懒的学习flexget，一直以来都是通过安装rutorrent
来充当rss下载器，然后qb监控ru的.session种子文件夹来实现自动下载。后来，我的这个方法受到了
严重挑战，一是不知道为何，qb在我的vps上难以修改watch folder，多番测试，结果都很玄学，可能与
配置文件的权限、是否先启动qb完成同意协议的过程并生成配置文件、浏览器缓存等等有关，试了好多次
后，最终放弃;二是ru实在太卡了，即使只负责下载种子，作为一个强迫症患者，删除种子的时候都会卡死，
是可忍孰不可忍，叔叔能忍，婶婶不能忍，那就ByeBye啦！

被逼无奈，最终捡起flexget，然后开启了新世界的大门！这玩意儿真的是非常便捷强大，比在rutorrent
里写正则表达过滤方便太多了！！

最后要介绍的PT贴吧大神做的autoremove-torrents，真的是神器，最终实现了全自动无人值守的盒子配置。
下面进入正题吧：
<!-- more -->
（本文所有命令均在Ubuntu 16.04 x64下运行，理论上讲18.04也可以，14.04主要是apt和
apt-get的区别,均在root用户下，/root/目录下运行）

## 安装qbittorrent

### 添加源
```shell
apt install software-properties-common python-software-properties
add-apt-repository ppa:qbittorrent-team/qbittorrent-stable
apt update && apt install qbittorrent-nox
```
### 设置自启动

```shell
nano /etc/systemd/system/qbittorrent.service
```

填入如下内容：

```shell
[Unit]
Description=qBittorrent Daemon Service
After=network.target

[Service]
User=root
ExecStart=/usr/bin/qbittorrent-nox
ExecStop=/usr/bin/killall -w qbittorrent-nox

[Install]
WantedBy=multi-user.target
```

使用方法：

```shell
systemctl enable qbittorrent  #开启自启动
systemctl start qbittorrent   #启动qb
systemctl stop qbittorrent    #停止qb
systemctl restart qbittorrent #重启qb
```

启动后应该可以在http://你的ip:8080/访问webui，用户名admin，密码adminadmin，登录后
切记修改用户名和密码！！！

## 安装flexget

### pip安装

```shell
apt install python3-setuptools python3-pip
pip3 install --upgrade pip
pip3 install flexget ##也可能是pip install，取决于你是否安装了python2.
```

### 填写flexget配置文件：

```shell
mkdir .flexget
nano .flexget/config.yml
```

填入以下内容：

```shell
templates:
  freespace:
    free_space:
      path: /home/star
      space: 10240     
  qb:
    qbittorrent:
      path: /home/star/Downloads/
      host: localhost
      port: 8080
      username: xx
      password: xx	  
  size:
    content_size:
      min: 10000
      max: 66666
      strict: no
tasks:
  CHDbits:
    rss: https://chdbits.co/torrentrss.php?rows=
    verify_ssl_certificates: no
    accept_all: yes
    template:
      - freespace
      - qb
    qbittorrent:
      label: CHDbits
  CMCT:
    rss: https://hdcmct.org/torrentrss.php?rows=
    verify_ssl_certificates: no
    accept_all: yes
    template:
      - freespace
	  - size
      - qb
    qbittorrent:
      label: CMCT
schedules:
  - tasks: [ChDbits, CMCT]
    interval:
      minutes: 10
```	  

以上配置是最简单的配置了，直接替换rss链接，就会下载10-66g之间的种子，每10分钟更新
一次，更多配置可以参考：
1. [flexget.config.yml](https://raw.githubusercontent.com/Aniverse/inexistence/968bf4ef1c92db5d53adf840ececd2e3aab97e89/00.Installation/template/config/flexget.config.yml)
2. [qbittorrent配合flexget](https://www.2dan.cc/pt/qbittorrent-flexget.html)
3. google和百度

### 测试flexget运行

```shell
flexget --test  --loglevel debug execute
```

看看是不是符合你的预期？

### flexget以daemon运行

```shell
flexget daemon start -d
```

### flexget以daemon开机自启动

```shell
nano /etc/rc.local
```

在exit0之前填入：

```
flexget daemon start -d
```



## 安装autoremove-torrents

软件来自[贴吧大神 @蛋糕机Mac](https://tieba.baidu.com/p/5617157829?red_tag=0564171599) 
2018年12月31号，最新版本1.2.4，但是我发现有重复统计的bug，就一直在用1.1了。

### 下载并解压

```shell
wget https://github.com/jerrymakesjelly/autoremove-torrents/archive/v1.1.0.tar.gz
tar zxf v1.1.0.tar.gz
cd autoremove-torrents-1.1.0
python3 setup.py install
```

### 在/root/下建立config.yml

```shell
nano config.yml
```

填入如下内容：

```
my_task: 
  client: qbittorrent
  host: http://127.0.0.1:8080
  username: XXX
  password: XXX
  strategies:
    s1:
      all_categories: true
      #seeding_time: 1209600
      seed_size:
        limit: 1020
        action: remove-old-seeds
  delete_data: true
```

这个设置是说，检查所有的分类，如果种子总大小超过1020G则删除较旧的种子和数据。
更多设置请参考[AMT-wiki](https://github.com/jerrymakesjelly/autoremove-torrents/wiki)

### 测试AMT运行

```shell
autoremove-torrents --view
```

看看是不是符合你的预期？

### 自动运行

```shell
crontab -e
*/15 * * * * /usr/local/bin/autoremove-torrents --conf=/root/config.yml ##切记按照这个写，原作者的那个bin文件位置不对
```

每十五分钟运行一次，看看有没有符合条件需要删除的种子。

## 监控qb和flexget的情况

### qb监控脚本

```shell
nano listen-qb.sh
```

填入：
```
#!/bin/bash
pgrep qbittorrent &> /dev/null
if [ $? -gt 0 ]
then
echo "`date` qbittorrent is stop"
systemctl start qbittorrent
else
echo "`date` qbittorrent running"
fi
```

### flexget监控脚本

```shell
nano listen-flex.sh
```

填入：
```
#!/bin/bash
pgrep flexget &> /dev/null
if [ $? -gt 0 ]
then
echo "`date` flexget is stop"
flexget daemon start -d
else
echo "`date` flexget is running"
fi
```

### 修改权限

```
chmod 777 listen-qb.sh
chmod 777 listen-flex.sh
```


### 定时运行

```shell
crontab -e
```

加入如下两行
```
*/5 * * * * /root/listen-qb.sh
*/5 * * * * /root/listen-flex.sh
```

每五分钟自动检测qb和flex是否再运行，没有则启动。

## 以上
