---
title: 如何解决 Windows 下执行 fastboot 出现 Press any key to shutdown 的错误
date: 2020-03-21 20:19:33
categories: Tips
tags:
    - Android
    - fastboot
    - Windows
---

在 Windows 上使用USB3.0运行 fastboot 时可能会遇到这个问题<!-- more -->
手机成功进入 fastboot 了，但在 cmd 执行`fastboot devices`后手机却断开了，屏幕上显示一行 Press any key to shutdown
而在一些预装 Win8 的机子上使用 USB3.0 却不会出现这样的问题

解决方法如下：

1. 手机进入 fastboot 模式，打开设备管理器，在详细信息选项卡查看手机的硬件ID，
2. 打开注册表编辑器，定位到`HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\usbflags`
3. 根据`VID` `PID` `REV`后的值找到对应的项（本例中为`18D1D00D0100`）
4. 按照这个值，新建一个类型为DWORD，名称为`SkipBOSDescriptorQuery`的数据
5. `数值数据`设为1，然后单击确定
6. 退出注册表编辑器，拔下并重新插入设备，以使解决方法生效

最终效果：

{% asset_img VIDPIDREV.png 最终效果 %}

> 当一些USB设备连接到EHCI的端口时，在Windows 8.1上可能无法枚举，但在Windows 8上可以工作。
> 在Windows 8.1中，该错误在设备管理器中报告为错误代码43。 原因之一是设备报告自己支持的USB版本大于2.00，但未提供所需的BOS描述符。
> 根据官方USB规范，版本大于2.00的USB设备必须提供BOS描述符。
> 在Windows 8中，USB 2.0驱动程序堆栈无法验证该要求。 因此，连接到EHCI控制器时，具有大于2.00版本且没有BOS描述符的设备将成功枚举。
> 在Windows 8.1中，驱动程序堆栈已更新，此类设备的枚举失败。
> 注意：Windows 8和8.1中的USB 3.0可扩展主机控制器（xHCI）驱动程序可验证该要求。

看来是Windows驱动的一个历史遗留问题

---

参考

[Windows 8.1 "A request for the USB BOS descriptor failed" - SOLVED !](https://forum.xda-developers.com/showpost.php?p=55529498&postcount=56)

[Why does my USB device work on Windows 8.0 but fail on Windows 8.1 with code 43?](https://web.archive.org/web/20190109023258/https://blogs.msdn.microsoft.com/usbcoreblog/2013/11/25/why-does-my-usb-device-work-on-windows-8-0-but-fail-on-windows-8-1-with-code-43/)