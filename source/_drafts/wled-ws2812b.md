---
title: 使用 ESP8266 搭建 WLED project
date: 2022-08-06 13:29:41
tags: 
    - WLED
    - ESP8266
---

制作一个通过 WIFI 控制的 LED 灯带
<!-- more -->

---

## 准备材料

| | |
| - | - |
| ESP8266 | ESP32支持2.4 GHz 和 5 GHz，ESP8266只支持 2.4 GHz |
| ws2812b led灯条 | 2米 |
| 杜邦线 | 连接 LED 灯带与 ESP |
| 快装压板 | 连接电源线及杜邦线 |
| DC 1.5 圆口母头 |  |
| USB 转 DC 1.5 圆口公头转换头 |  |
| 电源 | 一个常见的 5V3A DC USB 电源即可 |

## 刷入固件

1. 下载 [esptool](https://github.com/espressif/esptool/releases)

    解压出 `esptool`

2. 下载 [WLED](https://github.com/Aircoookie/WLED/releases)

    选择以 `ESP8266.bin` 结尾的文件

3. 使用 micro USB 数据线连接，并使用 esptool 将固件刷入 ESP

    `esptool write_flash 0x0 ./WLED_0.13.1_ESP8266.bin`

4. 连接 WLED

    使用 WiFi 设备连接 `WLED-AP`，默认密码 `wled1234`

## 连接灯带



### 连接方式

| 针脚 | 作用 | 接线颜色 |
| - | - | - |
| +5V | 电源 | 红 |
| Di | 数据 | 绿 / 白 |
| G | 接地 | 黑 |

### 功耗计算

USB 不使用 micro USB 供电

一个最高亮度的白光 LED 的电流为 60 mA

比如 100 颗 LED，可计算出最大功耗为 5V × 60 mA × 100 = 30 W

---

参考

- [WLED Project](https://kno.wled.ge/)
