---
title: 如何在twrp下升级MIUI
date: 2019-06-10 10:02:22
categories: Tips
tags:
    - Android
    - MIUI
---

miui更换第三方recovery之后无法进行OTA升级，MIUI系统更新包会对下载完成的增量包进行加密。
若要进行增量升级则需获得未经加密的增量包，然后再手动刷入。<!--more-->

---

1. 下载 [Brotli](https://github.com/google/brotli/releases) 解压`system.new.dat.br`文件

    ```cmd
    .\brotli.exe -d system.new.dat.br -o system.new.dat
    ```

2. 使用 [sdat2img](https://github.com/xpirt/sdat2img) 生成刷入的镜像

    ```cmd
    .\sdat2img.py system.transfer.list system.new.dat system.img
    ```

3. 进入twrp，因为是同内核升级，双清(Cache, Dalvik)即可，然后刷入system分区


---

参考

[[TUT] How to convert system.new.dat.br to system.new.dat](https://forum.xda-developers.com/android/help/tut-how-to-convert-dat-br-to-dat-t3723926)

[xpirt/sdat2img: Convert sparse Android data image to filesystem ext4 image](https://github.com/xpirt/sdat2img/blob/master/sdat2img.py)

