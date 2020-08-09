---
title: Padavan 固件 - 刷入及使用
date: 2020-03-25 02:05:00
update: 2020-05-03 16:12:00
comments: true
tags: 
- Linux
- 路由器
categories: [Linux, 路由器]
---

## 简介

在上一篇中，我写了小米路由器 3 刷 X-WRT 的操作；而在路由器的范畴里，另有一个众人皆知、出类拔萃的固件当属 padanvan 。

### 特点

- 稳定性：一个月不重启是常态。
- 性能强：芯片可以得到真正发挥。
- 易用性：与其他第三方固件相比更容易上手，管理页大部分信息简单又详细。
- 功能多：什么 KMS，软加速，SSR，Clash，去广告应有尽有。

<!-- more -->

本刷入方法理论与绝大多数已支持的机型通用。
刷入的固件为 hiboy 的 padavan 。

---

## 刷入 Breed

Breed 是一个 Bootloader
[原贴](https://www.right.com.cn/forum/thread-161906-1-1.html) [下载链接](https://breed.hackpascal.net)

下载你的路由器型号所对应的 Breed
**建议先在原帖查看个别机型的注意事项**

将 Breed 上传至路由器的 /tmp 目录下：（在本地命令行执行）
<font color=grey size=2>如未开启 SSH ，百度“路由器型号+开启ssh教程”</font>

```bash
scp <breed_file_path> root@<ip>:/tmp/breed.bin
#执行后会让你输入 SSH 密码
```

其中 `<breed-file-path>` 为下载好的 Breed 的路径， `<ip>` 为路由器管理IP。

将 Breed 刷入 Bootloader：

```bash
ssh root@<ip>
#先使用 SSH 连接到路由器
mtd -r write /tmp/breed.bin Bootloader
```

刷入完成后会自动重启

## 进入 Breed Web 控制台

<font color=grey size=3>在 breed 下是没有无线信号的，必须要提前准备网线将电脑与路由连接。</font>

步骤：

1. 在命令行中输入 `ping <ip> -t`

2. 拔掉电源

3. 按住复位键或 WPS 键不松，同时插入电源

4. ping 通之后即可松开按键

5. 浏览器输入 `192.168.1.1` ，即可进入 breed 控制台

## 刷入 Padavan

Padavan 固件 [更新贴](https://www.right.com.cn/forum/thread-1515729-1-1.html) 、[下载链接](https://opt.cn2qq.com/padavan/)

1. 下载你的路由器型号所对应的固件

2. 在 Breed Web 恢复控制台中，选择 固件更新 > 固件 ，并选择下载好的固件

3. 刷入完成后，重启路由器即可。

>默认密码：1234567890
>固件管理界面：192.168.123.1
>管理界面账户密码：admin/admin
