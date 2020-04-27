---
title: php优化设置
layout: post
categories:
- Tutorial
tags:
- linux
- php
---

之前折腾个人的小破站，搜集了不少各路“优化”设置，放在这里，以备不时之需啦。

<!-- more -->

# unix socket还是tcp sock？

## TCP socket

1. 编辑/etc/nginx/sites-available/default：

	```shell
	location ~ \.php$ {
		  index index.php index.html index.htm;
		  include /etc/nginx/fastcgi_params;
		  fastcgi_pass 127.0.0.1:9000;
		  fastcgi_index index.php;
		  include fastcgi_params;
	 }
	```
2. 编辑/etc/php5/fpm/pool.d/www.conf中的参数listen：

	```shell
	listen=127.0.0.1:9000
	```

##  unix socket

1. 创建socket接口位置

	```shell
	sudo touch /dev/shm/fpm-cgi.sock
	sudo chown www-data:www-data /dev/shm/fpm-cgi.sock
	sudo chmod 666 /dev/shm/fpm-cgi.sock
	```

	dev/shm目录是所谓的tmpfs，是RAM可以直接使用的区域，读写速度会快一些。

2. 编辑/etc/php5/fpm/pool.d/www.conf中的参数：

	```shell
	listen=/dev/shm/fpm-cgi.sock

	; Set listen(2) backlog. A value of '-1' means unlimited.
	; Default Value: 128 (-1 on FreeBSD and OpenBSD) 
	; 内存积压，默认是128，并发高了之后就会报错,根据自己内存设置
	listen.backlog = 128
	```

3. 编辑/etc/nginx/sites-available/default：

	```shell
	location~\.php${
		  indexindex.phpindex.htmlindex.htm;
		  include/etc/nginx/fastcgi_params;
		  fastcgi_passunix:/dev/shm/fpm-cgi.sock;
		  fastcgi_indexindex.php;
		  includefastcgi_params;
	}
	```

4. 修改/etc/sysctl.conf配置，提高内核级别的并发连接数

	```shell
	net.core.somaxconn = 2048

	sudo sysctl -p
	````

5. 重启nginx和php-fpm

## 两种区别

>从原理上来说，unix socket方式肯定要比tcp的方式快而且消耗资源少，因为socket之间在nginx和php-fpm的进程之间通信，而tcp需要经过本地回环驱动，还要申请临时端口和tcp相关资源。
当然还是从原理上来说，unix socket会显得不是那么稳定，当并发连接数爆发时，会产生大量的长时缓存，在没有面向连接协议支撑的情况下，大数据包很有可能就直接出错并不会返回异常。而TCP这样的面向连接的协议，多少可以保证通信的正确性和完整性。
当然以上主要是半懂不懂的理论分析加主观臆测，具体的差别还是要通过测试数据来说话，以后有空，会进行这方面的测试。从网上别人博客的测试数据，我的理论分析差不多是对的。至于你选择哪种方式，我只能说“鱼和熊掌不可兼得也”，通过高超的运维和配置技巧，在性能和稳定性上做一个平衡吧。

>楼主，你确定socket比tcp要好吗？我之前也是这么认为的，但是今天做压测的时候就发现tcp比socket稳定多了 同样的并发，超过某个数量的时候，socket就一直报：connect()to unix:/dev/shm/php55-socket failed (11: Resource temporarily unavailable。。。报错率大概是15%的样子，但是查看一些资料发现，tcp比socket这方面稳定，故改tcp，发现基本没有报错了。同样的并发下，起码出现502的几率相对减少，所以我还是改tcp了。

# php-fpm进程数的优化 [php-fpm进程数设置多少合适](https://zhuanlan.zhihu.com/p/94627701)

## 了解参数 


需要了解的参数是：pm、pm.max_children、pm.start_servers、pm.min_spare_servers、pm.max_spare_servers。

pm：表示使用那种方式，有两个值可以选择，就是static（静态）或者dynamic（动态）。

在更老一些的版本中，dynamic被称作apache-like。这个要注意看配置文件的说明。

下面5个参数的意思分别为：

```shell
pm = dynamic 如何控制子进程，选项有static和dynamic    
pm.max_children：静态方式下开启的php-fpm进程数量    
pm.max_requests：php-fpm子进程能处理的最大请求数    
pm.start_servers：动态方式下的起始php-fpm进程数量
pm.min_spare_servers：动态方式下的最小php-fpm进程数
pm.max_spare_servers：动态方式下的最大php-fpm进程数量
```

## 区别：

如果dm设置为 static，那么其实只有pm.max_children这个参数生效。系统会开启设置数量的php-fpm进程。

如果dm设置为 dynamic，那么pm.max_children参数失效，后面3个参数生效。

系统会在php-fpm运行开始 的时候启动pm.start_servers个php-fpm进程，

然后根据系统的需求动态在pm.min_spare_servers和pm.max_spare_servers之间调整php-fpm进程数。

在高并发场景（我们公司是一万并发测试）中，一般都是采用的static方式，因为一般来说我们的服务器都是有严格的业务分离，fpm服务器就是用来作为fpm服务器使用，除去还有一个nginx之外，不会去考虑有其他的服务器占用资源。

在php的运行中，无非是两种场景


1. 大运算的场景，即 php程序需要用大量的cpu资源来进行数据计算之类的操作，在这种场景下，fpm进程可以设置为cpu数量的一倍或者两倍
2. 高io场景，php的使用场景中（最起码是本电商场景中）基本上属于高io，因为程序花了大量的时间在等待redis返回等待数据库返回。高io场景下，因为cpu大多处在wa状态下，所以可以尽量的加大fpm进程数，所以这个时候使用内存/30m是更为合理的

经过我们自己真实压测，大量redis和mysql读写的io密集情况下，16G的内存，fpm我们设置为400个的时候qps比fpm 16个 32个要好不少

# php.ini调优

## 内存

php.ini中的 memory_limit设置用于设定单个 PHP 进程可以使用的系统内存最大值。

## 文件上传

```shell
file_uploads = 1
upload_max_filesize = 10M
max_file_uploads = 3
```

默认情况下，PHP允许在单次请求中上传20个文件，上传的文件最大为2MB，这里为单次请求最多只能上传3个文件，每个文件最大为10MB，这个值不要设置太大，否则会出现超时。

如果非要上传大文件，Web 服务器的配置也要做相应调整。除了在 php.ini 中设置之外，还要调整 Nginx 虚拟主机配置中的 client_max_body_size 设置。

## 最长执行时间

最长执行时间

php.ini 文件中的 max_execution_time 用于设置单个 PHP 进程在终止之前最长可运行时间。这个设置默认是 30 秒，建议将其设置为 5 秒：

```shell
max_execution_time = 5
```

在 PHP 脚本中可以调用 set_limit_time() 函数覆盖这个设置。


## 其他参见 

[php.ini 配置调优 —— 让 PHP 应用性能维持在更高水平](https://blog.csdn.net/dtfly520/article/details/76849855)

# Ref:

- [nginx和php-fpm通信使用unix socket还是TCP，及其配置](https://blog.csdn.net/pcyph/java/article/details/46513521)

- [php-fpm进程数设置多少合适](https://zhuanlan.zhihu.com/p/94627701)

- [php.ini 配置调优 —— 让 PHP 应用性能维持在更高水平](https://blog.csdn.net/dtfly520/article/details/76849855)



