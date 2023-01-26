---
title: 使用 ESP8266 搭建 WLED project
date: 2022-08-06 13:29:41
categories: Tips
tags: 
    - WLED
    - ESP8266
---

制作一个通过 WIFI 控制的 LED 灯带，布置于显示器后方，改善环境光
<!-- more -->

---

<img src="https://kno.wled.ge/assets/images/ui/headers/wled_logo_akemi.png" class="full-image" />

## 准备材料

| | |
| - | - |
| ESP8266 | CH340G/CP2102 |
| WS2812B LED 灯条 | 2米 |
| LED 3P公母对接线 | 连接每段 LED |
| 公对母杜邦线 | 连接 LED 灯带与 ESP |
| 一进二出对接头 或 电工胶带 | 连接电源线及杜邦线 |
| DC 5.5*2.5MM 转接母头 |  |
| USB 转 DC 2.5 公头电源线 |  |
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

| 针脚 | 接线颜色 | 作用 |
| - | - | - |
| Vin / +5V | 红 | 正极 |
| D4 / Di | 绿 | 数据 |
| GND / G | 白  | 负极 |

连接时需要注意电流方向，在灯带上会有标明

### 供电

一颗 WS2812B 灯珠 的最大电流为 60 mA，如有 100 颗 LED，最大功率为 5V × 60 mA × 100 = 30 W。

因此当灯珠较多时，单纯地使用 USB 供电无法稳定的输出最高亮度的白光。

一般而言，一个常见的 5V3A 充电头就可以点亮50颗左右的 LED 灯珠。

### 电压衰减

灯珠在 DC 电源远端可能会变暗，需要将两端灯带上的备用电源线相连接

---

参考

[WLED Project](https://kno.wled.ge/)
