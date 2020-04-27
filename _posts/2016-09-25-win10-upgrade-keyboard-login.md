---
layout: post
title: win10升级导致的键盘失灵和卡锁屏界面
categories:
  - ECM
tags:
  - win10
  - error
---

1. 升级win10后键盘失灵，必须重启才能使用。

网上的部分解决方案说关闭电源管理中的快速启动，个人又想保留这一功能，后各种查询，解决方法如下：

<!-- more -->

注册表删除：

[HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\PECKbdProtector]

文件删除windows/system32/driver/peckp_x64.sys

即可。

2. win10周年更新，卡锁屏界面的解决方案

打开管理员的命令提示符，运行
if exist "%SystemRoot%\System32\InputMethod\CHS\ChsIME.exe" (
takeown /f "%SystemRoot%\System32\InputMethod\CHS\ChsIME.exe" /A
icacls "%SystemRoot%\System32\InputMethod\CHS\ChsIME.exe" /deny "NT AUTHORITY\SYSTEM:RX"
)

就不会出现SYSTEM用户的ChsIME.exe了

参考：http://bbs.pcbeta.com/viewthread-1706070-1-1.html

还有人说关掉个性化-锁屏界面-在登录屏幕上显示锁屏界面背景图片，这个没有是不知道了。还有人说跟快速启动有关。。。为啥老是快速启动背锅啊。。。
