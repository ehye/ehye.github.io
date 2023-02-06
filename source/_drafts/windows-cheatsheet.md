---
title: Windwos Cheatsheets
categorties: Note
tags:
    - Windows
---


- 删除本地连接2

```
计算机\HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Network\{4D36E972-E325-11CE-BFC1-08002BE10318}
```

- 删除网络2

```
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles
```

- 重置系统代理

*解决 Epic 更新 Xbox live 登录*

```
netsh winhttp reset proxy
```
