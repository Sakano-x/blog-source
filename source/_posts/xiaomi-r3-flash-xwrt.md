---
title: 小米路由器 3 刷入 X-WRT 固件
date: 2020-03-20 12:45:00
update: 2020-05-03 16:16:00
comments: true
tags:
- Linux
- 路由器
categories: [Linux, 路由器]
references:
- name: OpenWrt Wireless 官网
  url: https://openwrt.org/toh/xiaomi/mir3?s[]=xiaomi
- name: X-WRT 官方贴
  url: https://www.right.com.cn/forum/thread-212965-1-1.html
---

## 前言

对于小米路由器 4 以前的机型，官方固件的优化都不够好；功能少，稳定性不高，断流的情况也时常发生，而且穿墙性能较差。有一天，我终于受不了了，在网上寻找了许多关于路由器刷机的教程。既然手机能刷机，那咱们给路由器也刷个机**吧**！

<!-- more -->

## 声明

- 本文参考了 [OpenWrt Wireless 官网](https://openwrt.org/toh/xiaomi/mir3?s[]=xiaomi) 和 [X-WRT 官方贴](https://www.right.com.cn/forum/thread-212965-1-1.html)，并对其部分步骤加以整理。
- 此教程**仅适用于小米路由器 3** ，并且未在其他型号的路由器上做过测试。
- 建议刷机前先熟悉 Linux 命令
- **<font color=red size=4>刷机有风险，刷机需谨慎！</font>**

---

## 解锁 SSH 权限

这一步有两种方法，第一种是降级后利用漏洞进行 HTTP 渗透攻击；第二种是在官网申请 SSH 并刷入工具包。下面我会讲详细操作，任选一种方法即可，**推荐使用方法一**，方便。

### 方法一

下载 [2.11.20 降级包](https://eduinhk-my.sharepoint.com/:u:/g/personal/lino_kakeri_eduinhk_onmicrosoft_com/EXOTO-H9ZzhLmtybqUg6ZggBGmT9miyPOzEHLl2_DOn-Lg?e=KloiPx) ，在路由器管理页面中 常用设置 > 系统状态 选择手动升级

过完设置向导后，正常登录管理页面，打开地址栏，你会看到形如 `http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/web/home#router` 的文本，将这里 `<STOK>` 的值记录下来。

依次访问下面这三条 URL ，请注意将 URL 中 `<STOK>` 的值替换为刚刚记录下来的

```bash
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/xqnetwork/set_wifi_ap?ssid=Xiaomi&encryption=NONE&enctype=NONE&channel=1%3Bnvram%20set%20ssh%5Fen%3D1%3B%20nvram%20commit
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/xqnetwork/set_wifi_ap?ssid=Xiaomi&encryption=NONE&enctype=NONE&channel=1%3Bsed%20%2Di%20%22%3Ax%3AN%3As%2Fif%20%5C%5B%2E%2A%5C%3B%20then%5Cn%2E%2Areturn%200%5Cn%2E%2Afi%2F%23tb%2F%3Bb%20x%22%20%2Fetc%2Finit.d%2Fdropbear
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/xqnetwork/set_wifi_ap?ssid=Xiaomi&encryption=NONE&enctype=NONE&channel=1%3B%2Fetc%2Finit.d%2Fdropbear%20start
```

在访问每一条 URL 后，正常情况都会显示 `{"msg":"未能连接到指定WiFi(Probe timeout)","code":1616}`

接下来，访问下面这条 URL ，`<OLDPWD>` 为登录路由器管理页面的密码，`NEWPWD` 为待会使用 SSH 连接的新密码。

```bash
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/xqsystem/set_name_password?oldPwd=<OLDPWD>&newPwd=<NEWPWD>
```

访问后，显示 `{"code":0}` ，则表明 SSH 已成功开启。

### 方法二

在 [官网](http://www1.miwifi.com/miwifi_download.html) 下载开发版固件并在路由器管理页面中手动升级，之后在官网 [开启 SSH 工具](http://d.miwifi.com/rom/ssh) ，照着网页内的步骤刷入工具包即可。

## 通过 SSH 连接

首先，准备一个 SSH 工具，连接到路由器

```bash
ssh root@192.168.31.1
```

登录密码为刚刚设置的 `NEWPWD`

## 启用串行端口

依次输入以下命令
<font size=2>**非常重要，因为小米默认锁死串口，若不开启，刷机失败或者出现意外，无法救回**</font>

```bash
nvram set flag_last_success=1
nvram set boot_wait=on
nvram set uart_en=1
nvram commit
reboot
```

最后会重启

## 刷入固件

此步骤有两种方法，**方法一适用于从官方固件刷入 X-WRT ，方法二适用于从其他第三方固件刷入 X-WRT** 。若你正在使用的固件为官方固件，则使用方法一。

### 方法一

在 [X-WRT 下载页](https://downloads.x-wrt.com/rom/) 找到 Xiaomi Mi Router R3，下载文件末尾为 `kernel1.bin` 和 `rootfs0.bin` 的文件。

将文件上传至路由器的 /tmp 目录下：（在本地命令行执行）

```bash
scp <kernel1-file-path> root@192.168.31.1:/tmp/kernel1.bin
scp <rootfs0-file-path> root@192.168.31.1:/tmp/rootfs0.bin
```

其中 `<kernel1-file-path>` 和 `<rootfs0-file-path>` 为下载好的那两个文件的路径。

接下来，使用 SSH 连接到路由器，刷入固件

```bash
ssh root@192.168.31.1
mtd write /tmp/kernel1.bin kernel1
mtd write /tmp/rootfs0.bin rootfs0
reboot
```

### 方法二

在 [X-WRT 下载页](https://downloads.x-wrt.com/rom/) 找到 Xiaomi Mi Router R3，下载文件末尾为 `breed-factory.bin` 的文件。

将文件上传至路由器的 /tmp 目录下：（在本地命令行执行）

```bash
scp <breed-factory-file-path> root@192.168.31.1:/tmp/breed-factory.bin
```

其中 `breed-factory-file-path` 为下载好的那个文件的路径。

接下来，使用 SSH 连接到路由器，刷入固件

```bash
ssh root@192.168.31.1
mtd write /tmp/breed-factory.bin firmware
reboot
```

## 开始使用

等待重启完成，即可通过 Wi-Fi 连接到路由器

>默认 SSID ：X-WRT_XXXX
>默认密码 ：88888888
>固件管理界面：192.168.15.1
>管理界面账户密码：root/admin

更多关于固件的功能及使用方法可前往 [论坛专帖](https://www.right.com.cn/forum/thread-212965-1-1.html) 或 [开发博客](https://blog.x-wrt.com) 查看。

## 系统更新

在 [X-WRT 下载页](https://downloads.x-wrt.com/rom/) 找到 Xiaomi Mi Router R3，下载文件末尾为 `sysupgrade.bin` 的文件。

进入路由器管理页，选择 系统 > 备份/升级 > 刷写新的固件，将文件上传即可。
