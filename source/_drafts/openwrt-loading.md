---
title: openwrt_loading
date: 2018-10-18 22:18:37
categories: router
tags:
	- openwrt
	- LEDE
---


opkg print-architecture
看架构

按住reset开机 出现呼吸灯表示已进入uboot
web进不去则清理浏览器缓存，试试虚拟机交换器
错误固件导致无限重启别断电（会变半砖），按reset即可

kernel.bin 和 rootf.bin 打包为zip作为固件上传
 
发行版软件源

https://mirrors.tuna.tsinghua.edu.cn/lede/
https://downloads.lede-project.org/

src/gz base http://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.1/packages/mipsel_24kc/base
src/gz luci http://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.1/packages/mipsel_24kc/luci
src/gz routing http://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.1/packages/mipsel_24kc/routing
src/gz packages http://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.1/packages/mipsel_24kc/packages
src/gz telephony http://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.1/packages/mipsel_24kc/telephony