---
title: ubuntu_cheatsheet
tags:
	- Ubuntu
---

更新软件
sudo apt-get update        # Fetches the list of available updates
sudo apt-get upgrade       # Strictly upgrades the current packages
sudo apt-get dist-upgrade  # Installs updates (new ones)

创建用户
sudo adduser ubuntu

删除文件夹
sudo rm -rf folder

后台运行程序
nohup dotnet ArchiSteamFarm.dll &

查看进程
jobs
新建会话后查看进程
ps -ef

结束进程
kill pid

重启
sudo reboot