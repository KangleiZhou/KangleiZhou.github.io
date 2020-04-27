---
layout: post
title: windows下ftp服务器无法访问
categories:
  - ECM
tags:
  - windows
  - ftp
  - firewall
---
以前在win2003下的ftp在安装iis和FTP服务，设置后，即可使用。

最近的新买了一台windows server 2008的，流程走完竟然不能访问。搜索了一下：

<!-- more -->

windows2003和windows2008服务器端安装IIS自带的FTP功能，如果防火墙关闭才可以通过FTP正常访问，打开FTP则不能访问，解决办法如下：

1. 在windows2003服务器端设置：

控制面板 》系统和安全 》Windows 防火墙 》允许的程序”，在“允许另一个程序”中添加 “C:\WINDOWS\system32\inetsrv\inetinfo.exe”这个程序，添加完成即可

2. 在windows 2008/2012服务器端设置：

控制面板 》系统和安全 》Windows 防火墙 》允许的程序”，在“允许另一个程序”中添加 “C:\Windows\System32\svchost.exe”这个程序，添加完成即可。

步骤2应该是大部分问题之所在，win10也需要调整此处。

参考：http://kkworms.blog.51cto.com/540865/558477

http://www.myhack58.com/Article/sort099/sort0100/2015/58873.htm
