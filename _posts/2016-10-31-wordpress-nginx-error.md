---
layout: post
title: wordpress与nginx迁移之坑
categories:
  - ECM
tags:
  - wordpress
  - nginx
---

1. nginx 设置include时*.conf，但实际设置文件却没有加.conf...

2. nginx certs写的结尾crt和cert不同。。找不到

3. wp-config.php中的WP_DEBUG 改成ture可以看到具体的错误提示“error establishing a database connection”，实际上我是把用户名和密码设置错了。。。

4. 413 Request Entity Too Large. 在nginx.conf的http{}段， 增大nginx上传文件大小限制

<!-- more -->

设置允许发布内容为8M:

```
client_max_body_size 8M;
client_body_buffer_size 128k;
```
另外如果运行的是php，那么还要检查`php.ini`，这个大小`client_max_body_size`要和`php.ini`中的如下值的最大值一致或者稍大，这样就不会因为提交数据大小不一致出现的错误。

```
post_max_size = 8M
upload_max_filesize = 6M
```
修改完配置后，别忘记重新加载。

虽然这样修改能解决一般问题，相信大家都会问`post_max_size` 和 `upload_max_filesize` 之间关系是什么？

从字面含义看一个是上传文件最大值`upload_max_filesize`，一个是通过POST提交数据的最大值`post_max_size`，一般在php中用的是POST方式上传。

所以`php.ini`中设置 `memory_limit` ; `post_max_size` ; `upload_max_filesize`

当上传大文件时为了防止网速慢时超时现象，需要适当调大`max_execution_time`的值。

参考：http://www.nginx.cn/837.html
