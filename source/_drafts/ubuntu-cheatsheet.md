---
title: ubuntu_cheatsheet
tags:
	- Ubuntu
---

# 显示当前主机的信息
hostnamectl

# 更新软件
sudo apt-get update        # Fetches the list of available updates
sudo apt-get upgrade       # Strictly upgrades the current packages
sudo apt-get dist-upgrade  # Installs updates (new ones)

# 创建用户
sudo adduser ubuntu

# 加入用户组
sudo adduser <username> <group>

# 查看所有用户和用户组
cat /etc/passwd
cat /etc/group

# 删除文件夹
sudo rm -rf folder

# 解压到指定文件夹
unzip file.zip -d destination_folder

# 后台运行程序
nohup dotnet ArchiSteamFarm.dll &

# 查看进程
jobs

# 新建会话后查看进程
ps -ef

# 结束进程
kill <pid>

# 查看端口状态
netstat -ntlp | grep -v tcp6

# ufw 删除规则
ufw status numbered
ufw delete 2

# 重启
sudo reboot

# 启动 apache
sudo service apache2 start

# 终止 nginx
nginx -s stop

# 生成 pfx
openssl pkcs12 -export -out k4asf.pfx -inkey xxx.key -in xxx.crt

# 设置GOPATH
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
# 启动ss-go
shadowsocks-server -c go/shadowsocks/bin/config.json > go/bin/log &