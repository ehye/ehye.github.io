---
title: 安卓解包方法
date: 2019-06-10 10:02:22
categories: Tips
tags:
    - Android
---

更换第三方recovery之后无法进行OTA升级，若要升级则需手动刷入完整包<!--more-->
此时可以找到`system.new.dat.br`然后解包转换为可用adb刷入的img文件


下载 [Brotli](https://github.com/google/brotli/releases) 解压br文件

```bash
brotli.exe --decompress --in system.new.dat.br --out system.new.dat
```

使用 [sdat2img](https://github.com/xpirt/sdat2img) 解包 dat 文件

```bash
sdat2img.py system.transfer.list system.new.dat system.img
```


---

ref
> https://forum.xda-developers.com/android/help/tut-how-to-convert-dat-br-to-dat-t3723926
> https://github.com/xpirt/sdat2img/blob/master/sdat2img.py
> https://www.banxiayue.com/brotli.html
