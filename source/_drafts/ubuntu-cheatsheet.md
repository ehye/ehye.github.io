---
title: ubuntu_cheatsheet
tags:
	- Ubuntu
---

- 显示当前主机的信息

```bash
hostnamectl
```

- 更新软件

```bash
sudo apt-get update        - Fetches the list of available updates
sudo apt-get upgrade       - Strictly upgrades the current packages
sudo apt-get dist-upgrade  - Installs updates (new ones)
```

- 创建用户

```bash
sudo adduser ubuntu
```

- 加入用户组

```bash
sudo adduser <username> <group>
```

- 查看所有用户和用户组

```bash
cat /etc/passwd
cat /etc/group
```

- 删除文件夹

```bash
sudo rm -rf folder
```

- 解压到指定文件夹

```bash
unzip file.zip -d destination_folder
```

- 后台运行程序

```bash
nohup dotnet ArchiSteamFarm.dll &
```

- 查看进程

```bash
jobs
```

- 新建会话后查看进程

```bash
ps -ef
```

- 结束进程

```bash
kill <pid>
```

- 查看端口状态

```bash
netstat -ntlp | grep -v tcp6
```

- ufw 删除规则

```bash
ufw status numbered
ufw delete 2
```

- 重启

```bash
sudo reboot
```

- 启动 apache

```bash
sudo service apache2 start
```

- 终止 nginx

```bash
nginx -s stop
```

- 生成 pfx

```bash
openssl pkcs12 -export -out k4asf.pfx -inkey xxx.key -in xxx.crt
```

- 设置GOPATH

```bash
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

- 启动ss-go

```bash
go/shadowsocks/bin/shadowsocks-server -c go/shadowsocks/bin/config.json > go/shadowsocks/bin/log &
```

- scp 传文件

```bash
scp -P 2003 E:\xx.ipk root@192.168.1.1:/tmp
```

- 清空日志

```bash
echo "" > log
# echo "" >> file # 在末尾添加
```


---
