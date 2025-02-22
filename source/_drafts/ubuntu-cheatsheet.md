---
title: Ubuntu 常用命令
categories: Note
tags:
  - Ubuntu
---

Ubuntu 常用命令 <!-- more -->

---

# 系统

- 更新软件

  ```bash
  sudo apt update
  sudo apt upgrade
  ```

- 显示当前主机的信息

  ```bash
  hostnamectl
  ```

- 更改时区

  ```bash
  sudo timedatectl set-timezone Asia/Shanghai
  ```

- 重启

  ```bash
  sudo reboot
  ```

- 快捷方式(软连接)

  ```bash
  ln -s /mnt/c/Users/xxx ~/win10
  ```

- 查看服务

  ```bash
  service xxx status
  ```

- 启动服务

  ```bash
  service xxx start
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

- 后台运行程序

  - 不需要读取输入时

    ```bash
    nohup dotnet ArchiSteamFarm &
    ```

  - 需要读取输入时

    ctrl + z 暂停，使用 `htop` 继续

    ```
    配置账号

    chmod +x AFS

    输入密码、二次验证

    ctrl + z 挂到后台

    htop continue
    ```

# 用户

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

# 防火墙

- 查看端口状态

  ```bash
  netstat -ntlp | grep -v tcp6
  ```

- ufw 允许和拒绝

  ```bash
  sudo ufw allow 53/tcp
  sudo ufw allow 11200:11299/tcp
  sudo ufw deny 53/tcp
  ```

- ufw 删除规则

  ```bash
  sudo ufw status numbered
  sudo ufw delete 2
  sudo ufw delete allow "12345"
  ```

# 文件

- 清空日志

  ```bash
  echo "" > log
  ```

- 在末尾添加

  ```bash
   echo "xxx" >> file
  ```

- 删除文件夹

  ```bash
  rm -rf folder
  ```

- 下载文件

  ```bash
  curl -L -o v2ray-plugin https://github.com/shadowsocks/v2ray-plugin/releases/download/v1.3.2/v2ray-plugin-linux-amd64-v1.3.2.tar.gz
  ```

- 解压到指定文件夹

  ```bash
  tar -xf file.tar.gz -C destination_folder/
  ```

  ```bash
  unzip file.zip -d destination_folder
  ```

- 查看空间占用

  ```bash
  ncdu
  ```

  ```bash
  du -h --max-depth=1 directory/ | sort -h
  ```

  ```bash
  df -h
  ```

# 应用

- 检查 nginx 配置

  ```bash
  nginx -t
  ```

- 终止 nginx

  ```bash
  nginx -s stop
  ```

- 生成 pfx

  ```bash
  openssl pkcs12 -export -out k4asf.pfx -inkey xxx.key -in xxx.crt
  ```

- 从 .crt 转换到 .pem

  ```bash
  openssl x509 -in mycert.crt -out mycert.pem -outform PEM
  ```

- 设置 GOPATH

  ```bash
  export GOPATH=$HOME/go
  export PATH=$PATH:$GOPATH/bin
  ```

- 启动 ss-go

  ```bash
  go/bin/shadowsocks-server -c go/bin/config.json > go/bin/log &
  ```

- scp 传文件

  ```bash
  scp -P 2003 E:\xx.ipk root@192.168.1.1:/tmp
  ```

- pm2 启动程序并命名为 myApp

  ```bash
  pm2 start "npm run dev" --name myApp
  ```

---
