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

加入用户组
sudo adduser <username> <group>

删除文件夹
sudo rm -rf folder

解压到指定文件夹
unzip file.zip -d destination_folder

后台运行程序
nohup dotnet ArchiSteamFarm.dll &

查看进程
jobs
新建会话后查看进程
ps -ef

结束进程
kill pid
qj2md
重启
sudo reboot

启动 apache
sudo service apache2 start

终止 nginx
nginx -s stop