---
title: 在 Ubuntu 上利用 Nginx 搭建 ASP.Net Core 站点
date: 2017-09-17 19:14:43
categories: Web
tags:
	- Ubuntu
	- Nginx
	- .Net Core
---

这篇讲如何在AWS上的 Ubuntu 部署 .NET Core 站点<!--more-->

---

# 准备工作

## 创建应用

[安装 .NET Core](http://www.microsoft.com/net/core#linuxubuntu)

新建文件夹
```bash
mkdir razor
```
新建 razor 应用
```bash
cd razor/
dotnet new razor
```
## 发布打包
```bash
dotnet publish
```
还可以将`publish`文件夹移动到较浅路径，方便操作，比如移动到 `/var/`
```bash
mv /bin/Debug/netcoreapp2.0/publish/ /var/
```

# 设置反向代理
反向代理将终止 HTTP 请求并将其转发到 ASP.NET Core 应用程序
反向代理服务器可以减轻诸如提供静态内容、缓存请求、压缩请求和 HTTP 服务器的 SSL 终止等工作

{% asset_img kestrel-to-internet2.png %}
{% asset_img kestrel-to-internet.png %}

## 安装 Nginx

使用 apt-get 安装 Nginx
```bash
sudo apt-get install nginx
```

显式启动它

```bash
sudo service nginx start
```

## 设置 Nginx

打开 `/etc/nginx/sites-available/default` 添加文本

```nginx
server {
    listen 80;
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
> 此 Nginx 配置文件将传入的公共通信从端口80转发到端口5000

运行 `sudo nginx -t` 来验证配置文件的语法
测试成功后运行 `sudo nginx -s reload` 来应用更改

# 监听应用程序
Nginx 现在已可将外网访问 `http://yourhost:80` 的请求转发到在 `http://127.0.0.1:5000` 上运行的 ASP.NET Core 应用程序
但是 Nginx 未管理 [Kestrel](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore2x#when-to-use-kestrel-with-a-reverse-proxy)  进程

下面使用 systemd 创建一个服务文件来创建守护进程，以启动和监视 web 应用程序

##  创建服务文件

```
sudo vim /etc/systemd/system/kestrel-razor.service
```

按照 .net 应用路径设置

```ini
[Unit]
Description=Example .NET Web API Application running on Ubuntu

[Service]
WorkingDirectory=/var/publish
ExecStart=/usr/bin/dotnet /var/publish/razor.dll
Restart=always
RestartSec=10  # Restart service after 10 seconds if dotnet service crashes
SyslogIdentifier=dotnet-example
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production 

[Install]
WantedBy=multi-user.target
```
> 若有更改则需运行`systemctl daemon-reload`来重新载入配置

## 创建守护进程

启用服务
```bash
systemctl enable kestrel-razor.service
systemctl start kestrel-razor.service
```
验证运行
```bash
systemctl status kestrel-razor.service
```

{% asset_img status.png %}

如果有显示`Active: active (running)`就表明可以从本地和远程访问了

# 保护应用程序

## 设置防火墙（可选）
```bash
sudo apt-get install ufw
sudo ufw enable

sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

sudo ufw allow 22/tcp		# SSH
sudo ufw allow 8989/tcp		# SS
```
**注意要在这里开启你需要的所有端口**，这会覆盖AWS安全组