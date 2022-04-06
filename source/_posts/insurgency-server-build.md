---
title: 在 Ubuntu 上搭建 Insurgency 对战服务器
date: 2017-12-02 20:01:00
categories: Game
tags:
	- Steam
	- Ubuntu
	- Insurgency
---

{% asset_img Insurgency_2014.jpg %}

<!-- more -->

# SteamCMD

## 下载
1. 官方不建议使用管理员账户运行SteamCMD，因此建立一个叫做steam的用户
```
sudo useradd -m steam
```

2. 进入它的根目录
```
cd /home/steam
```

3. 安装包
```
sudo apt-get install steamcmd
```

4. 链接生成可执行文件
```
ln -s /usr/games/steamcmd steamcmd
```

## 运行
```
cd ~
steamcmd
```

## 登录
Insurgency 支持匿名登陆
```
login anonymous
```

# Insurgency

1. 设置安装路径

```text
force_install_dir /usr/games/ins-ds/
```

2. 查找服务器ID并下载

Insurgency 服务器ID为237410，可加`validate`来检验下载
```
app_update 237410 validate
```

3. 配置服务器
创建文件路径`/ins-ds/insurgency/cfg/server.cfg`写入以下配置
```
// ---------------------------------------------------------------
// Server Info Options
// ---------------------------------------------------------------
hostname "<YourServerName>"	// server name
rcon_password "<YourRemoteAdminPassword>"	// rcon password
sv_password ""		// Server password for private servers
sv_minrate 30000 // recommended minimum rate

// ---------------------------------------------------------------
// Server Download Options (Community made maps) 
// ---------------------------------------------------------------
// sv_downloadurl "<type-url-here>"
// sv_allowdownload 1
// sv_allowupload 1

// ---------------------------------------------------------------
// Server Logging Options
// ---------------------------------------------------------------
log on
sv_logbans 1
sv_logecho 1
sv_logfile 1
sv_log_onefile 0

// ---------------------------------------------------------------
// Game Mode Options (Change text with in the quotes)
// ---------------------------------------------------------------
// "mapcycle.txt" - by default this contains the most popular options
// "mapcycle_all.txt"  - all possible map/mode combinations for PvP
// "mapcycle_ambush.txt" - only ambush (VIP) mode
// "mapcycle_attackdefend.txt" - mix of attack/defend modes
// "mapcycle_cooperative.txt" - checkpoint, outpost, hunt
// "mapcycle_firefight.txt" - all firefight maps
// "mapcycle_flashpoint.txt" - all flashpoint maps
// "mapcycle_infiltrate.txt" - all infiltrate (CTF) maps
// "mapcycle_objrespawn.txt" - all modes featuring respawning for completing objectives
// "mapcycle_occupy.txt" - all occupy maps
// "mapcycle_push.txt" - all push maps
// "mapcycle_singlelife.txt" - mix of all single life modes
// "mapcycle_skirmish.txt" - all skirmish maps
// "mapcycle_workshop.txt" - used by Workshop system
mapcyclefile "mapcycle.txt"

// ---------------------------------------------------------------
// Enabling Matchmaking (Change text with in the quotes)
// More info: http://steamcommunity.com/app/222880/discussions/2/558746089590579609/
// ---------------------------------------------------------------
// "pvp" (Player vs Player)
// "custom" (Custom rules and modded servers)
// "coop" (Cooperative)
sv_playlist pvp
```

4. cd 到游戏目录建立以下脚本

```
./srcds_run -console -ip YourIP -port 27015 +map market_coop +maxplayers 8
```

4. sh 运行脚本

# 效果

<del>延迟一百多，估计是单核的原因吧</del>
服务器问题跳 ping
{% asset_img 20171202161412_1.jpg %}

---
# 参考

https://developer.valvesoftware.com/wiki/SteamCMD
https://developer.valvesoftware.com/wiki/Dedicated_Servers_List
https://developer.valvesoftware.com/wiki/Insurgency_2014_Dedicated_Server