---
title: OpenWrt折腾记录00
date: 2018-10-18 22:18:37
categories: Router
tags:
	- OpenWrt
---

记之前刷网件R6220的过程<!-- more -->

---

# 下载镜像

到openwrt官网[查找固件](https://openwrt.org/toh/views/toh_fwdownload)
下载*Firmware OpenWrt Install URL*中的`kernel.bin`和`rootfs.bin`，打包为zip作为固件来刷机

# 刷机
## 刷入不死bootloader

下载对应[Breed](https://www.right.com.cn/forum/thread-161906-1-1.html)放到U盘或开一个http服务器([HFS](http://www.rejetto.com/hfs/))

使用 telnet 登陆到路由器，浏览器打开网址 http://192.168.1.1/setup.cgi?todo=debug, 用户名密码默认 admin password
这时你应该会看到  "Debug Enabled!" 字样，启用Telnet成功。

进入到U盘文件夹，刷入bootloader
```
mtd_write write breed*.bin Bootloader
```
成功后就可以随意刷固件，出问题时reset即可

## 刷入固件
按住reset开机，出现呼吸灯表示已进入breed
> web进不去则清理浏览器缓存，试试删除虚拟机交换器，
 若刷错误固件导致无限重启别断电（会变半砖），按reset即可，
 第二次折腾时变半砖，靠[nmrpflash](https://github.com/jclehner/nmrpflash)救了回来。

将上一步kernel.bin 和 rootf.bin 打包的zip作为固件上传
 
# 配置软件源
首先看架构`opkg print-architecture`
```bash
arch all 1
arch noarch 1
arch mipsel_24kc 10
```
可以看到架构是mipsel_24kc，现在就可以到镜像网站找对应架构的源

有两个镜像站
- https://mirrors.tuna.tsinghua.edu.cn/lede/
- https://downloads.lede-project.org/

在国内就用清华的
```
src/gz base https://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.2/packages/mipsel_24kc/base
src/gz luci https://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.2/packages/mipsel_24kc/luci
src/gz routing https://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.2/packages/mipsel_24kc/routing
src/gz packages https://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.2/packages/mipsel_24kc/packages
src/gz packages_ex https://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.2/targets/ramips/mt7621/packages
src/gz telephony https://mirrors.tuna.tsinghua.edu.cn/lede/releases/18.06.2/packages/mipsel_24kc/telephony
```

其中第五行是目标类型为ramips，子类型为mt7621的补充package，注意名称不能一样

{% note warning %}

若https链接下载不了，先改成http，更新后再改为https

```
opkg update
opkg install wget # 更新 wget 支持 SSL
opkg install ca-certificates # 添加证书
opkg install libustream-openssl # 添加 SSL 库
```

{% endnote %}

连上网后就可以更新了
```
opkg update
opkg upgrade
opkg install <packages>
```

---

**ref**
https://www.right.com.cn/forum/thread-208580-1-1.html
https://emersion.fr/blog/2017/installing-lede-on-a-netgear-r6220/