---
layout: post
title: Ubuntu重启后SSH不启动以及Syncthing更新1.0后无法启动
categories:
  - ECM
tags:
  - Ubuntu
  - SSH
  - openvz
---

古老的Openvz终于露出了狰狞的獠牙，美人白发，英雄迟暮，不单单是世人看不下去，他们自己，也看不下去。。。

<!-- more -->

## 不知道原因就再也进不去的Ubuntu

Linux的用户，真的应该谨记，计算机是用来用的，不是用来折腾的，只要它还能动弹，就不要去碰它。今天闲的荒，apt upgrade出了事。。。不知道更新了什么，重启后就ssh不上了，当时服务器可以ping通，ISP的后台也能看到机器在运行，最终在ISP的后台开了online console，登上去看了下，ssh和syncthing都无法启动了，然后折腾了一下午，解决如下：

## SSH无法启动的解决

'sshd -t'后提示：
`Missing privilege separation directory: /var/run/sshd`

早在2006年就有人遇到过这个[问题了](https://bugs.launchpad.net/ubuntu/+source/openssh/+bug/45234)，当时的解决办法就是，`rc.local, bootmisc.sh - either way, it's seems trivial enough to me (yet I managed to get my patch wrong, aaargh!) and I hope this will be fixed soon.` 

比较悲伤，虽然我不知道是为啥，但十多年后我也遇到了。。。然后我又看到了[这个](https://talk.plesk.com/threads/missing-privilege-separation-directory-var-run-sshd.350438/),作者提问的同时也给出了解决办法：

```shell
crontab -e
@reboot mkdir -p -m0755 /var/run/sshd && systemctl restart sshd.service
```

其实就是重启强行重建这个文件夹。。简单粗暴，但在找不到原因，又不想重装的情况下，不失为一个很好的选择了。


## syncthing无法启动的解决

更新完syncthing的1.0版本后，无法启动了，这不是第一次发生了，这次的google终于有了些进展。

之前我用的systemd文件如下：
```
[Unit]
Description=Syncthing - Open Source Continuous File Synchronization for %I
Documentation=man:syncthing(1)
After=network.target
Wants=syncthing-inotify@.service

[Service]
User=%i
ExecStart=/usr/bin/syncthing -no-browser -no-restart -logflags=0
Restart=on-failure
SuccessExitStatus=3 4
RestartForceExitStatus=3 4

[Install]
WantedBy=multi-user.target
```

然后到github页查看发现，官方syncthing@.service的内容为：
```shell
[Unit]
Description=Syncthing - Open Source Continuous File Synchronization for %I
Documentation=man:syncthing(1)
After=network.target

[Service]
User=%i
ExecStart=/usr/bin/syncthing -no-browser -no-restart -logflags=0
Restart=on-failure
SuccessExitStatus=3 4
RestartForceExitStatus=3 4

# Hardening
ProtectSystem=full
PrivateTmp=true
SystemCallArchitectures=native
MemoryDenyWriteExecute=true
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```
但是用如上内容也无法启动syncthing，依旧启动失败，之后发现了如下两个issue:
1. [#5449](https://github.com/syncthing/syncthing/issues/5449)
2. [#5286](https://github.com/syncthing/syncthing/issues/5286)

之后把systemd文件中如下两项#掉，终于可以用了：
```
# SystemCallArchitectures=native
# MemoryDenyWriteExecute=true
```
issue中的对话值得深思：

```
@serl
Problem is, as you can guess from my version information, I cannot update the kernel to support these options, as the machine is a OpenVZ VPS and I have no control on the hypervisor.

Relates to #5286

Version Information
Syncthing Version: v1.0.0
OS Version: Ubuntu 16.04 / Kernel 2.6.32-openvz-042stab120.18-amd64
```

```
@calmh
Nonetheless, you’ve solved the problem. Outdated kernels and other software will always be around, but we cannot be forever prevented from using new features.
```

openvz不知道还能坚持多久？廉价vps如果没有了ovz的支持，会怎样走下去？毕竟2.6.32，真的太老了。。。