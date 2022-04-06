---
title: OpenWrt折腾记录01
date: 2019-01-15 12:28:41
categories: Router
tags:
	- OpenWrt
	- aria2
	- NFS
---
路由器+转接线+旧硬盘=私有云存储<!-- more -->

---

# 挂载移动硬盘

## 安装驱动

- 硬件驱动

```bash
opkg install kmod-usb-core
opkg install kmod-usb-uhci kmod-usb-ohci # USB 1.1
opkg install kmod-usb2
opkg install kmod-usb3 # 实测就算路由器上的接口是2.0，但用了3.0的线也要这个包
opkg install kmod-usb-storage # USB存储设备
opkg install kmod-usb-storage-extras # 读卡器
opkg install kmod-scsi-core # SCSI协议支持
opkg install kmod-scsi-generic
```

因为连接的是硬盘而不是U盘，连接线是某联的sata转usb线，走的是SCSI协议，因此还要额外安装 `kmod-usb-storage-uas`，网上帖子都没有提到，最后在官方文档找到

```bash
opkg install kmod-usb-storage-uas
```

- 文件系统驱动

```bash
opkg install kmod-fs-ext2
opkg install kmod-fs-ext3
opkg install kmod-fs-ext4
opkg install kmod-fs-ntfs
opkg install kmod-fs-vfat
```

## 挂载硬盘

安装挂载点

```
opkg install block-mount
```

新建目录

```cmd
mkdir /mnt/sda1
```

挂载硬盘

```bash
mount /dev/sda1 /mnt/sda1
```

> 使用Diskgenius将硬盘格式化为ext4，Windows上装Ext2Fsd即可像普通分区一样对ext4分区进行操作
> 卸载分区要先在Ext2Fsd里进行，最后在Windows正常卸载，否则不能关机

接上路由，使用mount命令或在挂载点进行挂载

```bash
mount -o anon \\192.168.1.1\mnt\sda1 Z:
```

# Aria2下载工具

## 安装

```bash
opkg install aria2 luci-app-aria2 luci-i18n-aria2-zh-cn
```

## 配置

- **以此用户权限运行**选`root`
- **附加选项列表**增加一项`check-certificate=false`，即可下载https
- **默认下载目录**选择硬盘的目录，例如`/mnt/sda1/`
- **指定日志位置**`log=/var/etc/aria2/aria2.log`
- **磁盘预分配**选择`Falloc` (ext4格式时)
- **添加额外的Tracker**，[附加 Bt tracker 列表](https://github.com/ngosang/trackerslist)

## 安装前端

这里选择 webui-aria2

```bash
opkg install webui-aria2
```

## 启用HTTPS

先为 uHTTPd 添加 https 证书
> https://openwrt.org/docs/guide-user/luci/getting-rid-of-luci-https-certificate-warnings

配置文件是每次自动生成的，因此直接修改无效，通过 UCI 增加参数反而更简单

```sh
rpc-secure=true
rpc-certificate=/etc/ssl/mycert.pem
rpc-private-key=/etc/ssl/mycert.key
```

证书要pem格式，就用uHTTPd的crt格式证书转换了

```
openssl x509 -in mycert.crt -out mycert.pem -outform PEM
```

端口都设置为6800，在webui勾选`启用 SSL/TLS 加密`，reload 之后就可以通过HTTPS访问了

# NFS服务

## 服务端

1. 安装nfs-kernel-server，这会自动下载所有需要的包

    ```
    opkg install nfs-kernel-server
    ```

2. 服务端配置

    编辑`/etc/exports`，指定匿名用户的uid和gid，方便Windows客户端访问

    ```
    /mnt/sda1       *(rw,no_root_squash,no_subtree_check,sync,insecure,anonuid=0,anongid=0)
    ```
    保存后执行`service nfsd reload`重启NFS

## 客户端（Windows10）

1. 启用或关闭Windows功能中启用NSF服务

2. 使用cmd挂载网络存储
    ```
    mount -o anon \\192.168.1.1\mnt\sda1 Z:
    ```
    计算机中出现了一个Z盘，此时访问会出现权限限制

3. cmd 运行 mount 命令
    ```cmd
    本地    远程                                 属性
    -------------------------------------------------------------------------------
    Z:       \\192.168.1.1\mnt\sda1                 UID=-2, GID=-2
                                                    rsize=16384, wsize=16384
                                                    mount=soft, timeout=10.0
                                                    retry=1, locking=yes
                                                    fileaccess=755, lang=GB2312-80
                                                    casesensitive=no
                                                    sec=sys

    ```
    记下uid和gid，Z盘属性-NFS属性，填入uid和gid

4. 使用UTF8编码，解决中文乱码
    [一个小设置，让Win10 NFS正常显示中文UTF-8](https://zhuanlan.zhihu.com/p/46254792)

    {% note danger %}
    注意：会导致部分中文软件会显示乱码
    {% endnote %}

## 客户端（Android）

手机可使用 [ES File Explorer](http://www.estrongs.com/)，安卓电视用 [Kodi](https://kodi.tv/)

---

参考

[OpenWrt Project: Installing and troubleshooting USB Drivers](https://openwrt.org/docs/guide-user/storage/usb-installing)

[OpenWrt Project: NFS Network File System share configuration (aka Linux/Unix file sharing)](https://openwrt.org/docs/guide-user/services/nas/nfs_configuration)