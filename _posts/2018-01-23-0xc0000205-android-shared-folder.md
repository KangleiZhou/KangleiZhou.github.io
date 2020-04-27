---
layout: post
title: android电视盒子无法打开win10共享文件夹
categories:
  - ECM
tags:
  - android
  - tvbox
  - win10
  - error
---
前段时间买了个android电视盒子，在家中服务器上开启了共享，之前没有问题，突然有天win10升级，之后就各种诡异。共享的盘符，随即出现一个或多个无法访问，具体表现就是共享文件夹点了没有反应，而同时win电脑则可以访问。换了android手机用es文件管理器同样无法访问，也没有错误提示。

后来想了个办法，整个文件夹复制到手机，终于看到了错误提示代码 (0xC0000205)。

<!-- more -->

查了无数资料，百度无望，都是在说改权限，但我的设置没有问题。最后在谷歌上搜了好久，一个[论坛回复](https://www.tenforums.com/network-sharing/43931-samba-access-android-device-stopped-working-post885824.html?s=62a07bacd1f654ea36105f0a2d60cbe7#post885824)给出了正解。

So follow the link and add the new `IRPStackSize DWORD` in

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters`

and set the value to `50` works for me. Hope this helps other people.

注意，50是十进制数

他注册论坛为了分享，我注册论坛，为了感谢他。

也希望能帮到需要的人吧.

---

2018.11.1 在“程序和功能”-“启动或关闭windows功能”-“SMB1.0XX”开启。

---

2019.5.3 another [link](https://forum.kodi.tv/showthread.php?tid=304203) may be helpful.

1. 

```
Click on Connections:Ethernet
Click on Properties button (lower left)
Double click internet Protocol Version 4 (TCP/IPv4)
Click Advanced Button lower right
Click the WINS tab
Click the radio button to Enable NetBios over TCP/IP
```

2.
 
```
I could not get the sc config command (or sc.exe in Powershell) to achieve the result I desired, so I resorted to two registry edits and a reboot.  See new steps below. Try this:
1. Run "tasklist /svc"
In the output, you should see something like this: Browser and LanmanServer running in their own svchost processes. *Sample output below is abbreviated for illustration purposes. You will see a longer list.Image Name                     PID Services
========================= ======== ==============
svchost.exe                   2772 LanmanServer
svchost.exe                   2868 Browser

2. Run these two commands to add the SvcHostSplitDisable registry value set to 1.

REG ADD HKLM\SYSTEM\CurrentControlSet\Services\Browser /v SvcHostSplitDisable /t REG_DWORD /d 1 /f

REG ADD HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer /v SvcHostSplitDisable /t REG_DWORD /d 1 /f

This will reconfigure the Browser service to run in a shared svchost with LanmanServer like in previous versions of Windows.
```

3. 

```
Create a user profile with password in Windows 10. It's not wise to use your main Windows admin profile as Kodi will leak this password in plain text.

Make sure your media folder is shared to this user. I will presume you know how to set up a Windows share.

Install Kodi v.18. Until Microsoft fixes SMB v.1 (maybe in June), you cannot share to earlier builds.

(More info: https://support.microsoft.com/en-us/help...-kb4103721 )

In Kodi > Settings > Services > SMB > Default settings (min none, max v.3).

In Kodi File Manager > Add Source > Browse > Add Network Location > (Protocol SMB) > Enter fields: Server Name, Shared Folder, Username, Password (do no attempt to browse there).

Once you've succeeded in creating a link to your shared folder you can choose it as an Add Source shortcut for all media.

NB: Make sure you instruct Kodi to remember your password.
```

4. 
**Don't use USB Hub to connect your hard driver!!**
