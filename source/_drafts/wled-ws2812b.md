---
title: 使用 ESP8266 搭建 WLED project
date: 2022-08-06 13:29:41
tags: 
    - wled
    - ESP8266
---

做一个通过WIFI控制的 LED 灯带
<!-- more -->

---

[WLED Project](https://kno.wled.ge/)


## 硬件


| Syntax      |  |
| ----------- | ----------- |
| ESP8266   | Text        |
| Paragraph   | Text        |
| ws2812b led灯条      | Title       |
| 杜邦线   | Text        |
| 电源   | Text        |


## 软件

### WLED

本地刷入方法

```bash
python get-pip.py
```

```bash
pip install esptool
```

从[release](https://github.com/Aircoookie/WLED/releases)页面下载固件

刷入
```bash
esptool.exe write_flash 0x0 ./WLED_0.13.1_ESP8266.bin
```

