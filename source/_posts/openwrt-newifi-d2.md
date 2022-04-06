---
title: OpenWrt折腾记录04
date: 2019-06-18 09:45:48
categories: Router
tags:
	- OpenWrt
---

捡了台新3（newifi-d2)，放弱电箱当有线路由吧<!-- more -->

---

## 刷入固件

1. 开启 SSH

    在浏览器中输入`http://192.168.99.1/newifi/ifiwen_hss.html`，页面显示`success`即表明已开启SSH

2. 上传解锁文件到路由器

    将解锁文件`newifi-d2-jail-break.ko`放U盘插上路由器

3. 开始解锁

    SSH执行`insmod newifi-d2-jail-break.ko`，成功后路由器会自动重启，断电后按复位健/USB键开进入Breed

## 配置

1. 设置静态地址

    在有线路由里配置好无线AP的静态路由，例如`192.168.1.2`

2. 连接无线路由

    将网线连接到无线AP的任一LAN口，无线AP的LAN口配置为`DHCP服务器`，接到WAN口则可配置为二级路由

## 结果

路由器IP依然为有线路由的IP

---

参考

[AR/QCA/MTK Breed，功能强大的多线程 Bootloader](https://www.right.com.cn/forum/thread-161906-1-1.html)
