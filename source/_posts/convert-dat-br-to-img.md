---
title: 安卓解包方法
date: 2019-06-10 10:02:22
categories: Tips
tags:
    - Android
---

miui更换第三方recovery之后无法进行OTA升级，因为系统更新会对下载完成的增量包进行加密。
若要进行增量升级则需获得未经加密的增量包，然后再手动刷入。<!--more-->

---

找到`system.new.dat.br`然后解包转换为可用adb刷入的img文件
1. 下载 [Brotli](https://github.com/google/brotli/releases) 解压br文件
```bash
.\brotli.exe -d system.new.dat.br -o system.new.dat
```
    {% note success %}
    可以利用自带下载管理来下载，从而绕过系统更新对完成的zip进行加密的操作
    - 系统更新点击下载
    - 下载管理点击暂停再点击继续
    {% endnote %}

    {% note info %}
    或者直接到miui官网下载最新全量包
    {% endnote %}

2. 使用 [sdat2img](https://github.com/xpirt/sdat2img) 解包 dat 文件
```bash
sdat2img.py system.transfer.list system.new.dat system.img
```

3. 进入twrp直接刷入system分区


---

ref
> https://forum.xda-developers.com/android/help/tut-how-to-convert-dat-br-to-dat-t3723926
> https://github.com/xpirt/sdat2img/blob/master/sdat2img.py
> https://www.banxiayue.com/brotli.html
