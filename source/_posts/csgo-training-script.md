---
title: 如何设置 CSGO 训练脚本
date: 2017-10-16 19:23:02
categories: Game
tags:
	- CSGO
---

新建一个文本文档，复制黏贴以下代码

```
sv_cheats 1
sv_showimpacts 1
sv_infinite_ammo 1
sv_grenade_trajectory 1
mp_roundtime 5
mp_buytime 9999
mp_buy_anywhere 1
mp_maxmoney 16000
mp_startmoney 16000
mp_freezetime 0
ammo_grenade_limit_total 5
mp_roundtime_defuse 60
bind alt noclip
mp_restartgame 1
```

另存为 cfg 文件到 CSGO 安装路径

如`X:\Program Files\Steam\steamapps\common\Counter-Strike Global Offensive\csgo\cfg\xxjj.cfg`

进入游戏，选择休闲/竞技模式

按`~`调出控制台，输入 `exec xxjj`

> CSGO 优先使用config.cfg 设定 bind 后会自动存档到 config.cfg
