---
title: How to set Shadowsocks over HTTPS With Nginx
date: 2020-03-27 18:53:20
categories: Tips
tags:
    - Nginx
    - Shadowsocks
    - SSL
    - V2Ray
---

如何将 Shadowsocks 流量伪装成 HTTPS 请求
<!-- more -->

---
借助 Nginx 并配合 V2Ray 插件，根据路由判断是否为梯子的数据，将其转发到梯子程序，则可实现 HTTPS 与梯子共用443端口

## 原料

- Shadowsocks-libev
- Nginx
- SSL 证书 [acme.sh](https://github.com/acmesh-official/acme.sh) （DNSpod 等免费 SSL 证书亦可）
- DNS解析
- V2Ray-plugin [release](https://github.com/shadowsocks/v2ray-plugin/releases)

## 配置 Nginx 及证书

1. 颁发 SSL 证书

    ```bash
    acme.sh --issue -d mydomain.com --nginx
    ```

2. 复制/安装

    不要让 Nginx 直接使用`~/.acme.sh/`下的文件，将生成的证书复制到`/etc/nginx/certs/`下后，配置 Nginx

    ```nginx
    server {
        listen 443 ssl default_server;
        #listen [::]:443 ssl default_server; # 没有 ipv6 可以不写这行
        ssl_certificate /etc/nginx/certs/fullchain.cer; # 证书位置
        ssl_certificate_key /etc/nginx/certs/mydomain.com.key; # 证书位置
        ssl_session_timeout 3m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;

        server_name mydomain.com;

        location /ray { # 经此路径流量将转发给梯子
            proxy_redirect off;
            proxy_pass http://127.0.0.1:8008;
            proxy_set_header Host $http_host;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
    }
    ```

3. 设置自动更新

    ```bash
    acme.sh --upgrade --auto-upgrade
    ```

## Shadowsocks 配置

- 安装 v2ray 插件

    将下载的 release 解压到 `/user/bin`

- 服务端配置

    `/etc/shadowsocks-libev/config.json`

    ```json
    {
        "server":"localhost",
        "server_port":8008,
        "mode":"tcp_only",
        "local_port":1080,
        "password":"***",
        "timeout":5,
        "method":"chacha20-ietf-poly1305",
        "plugin":"v2ray-plugin",
        "plugin_opts":"server;host=127.0.0.1;path=/ray"
    }
    ```

    {% note info %}
  - `server`设置为只允许从本地访问
  - `server_port`与`proxy_pass`一致
  - `path`与`location`一致
  - 因为是通过 Nginx 转发，同时也配置了HTTPS，所以`plugin_opts`不需要再填证书参数
    {% endnote %}

- Windows 客户端配置

    编辑配置文件`gui-config.json`

    ```json
    {
        "server": "mydomain.com",
        "server_port": 443,
        "password": "***",
        "method": "chacha20-ietf-poly1305",
        "plugin": "v2ray-plugin.exe",
        "plugin_opts": "tls;host=mydomain.com;path=/ray",
        "timeout": 5
    }
    ```

## DNS解析

最后一步，使用DNS解析服务，将你的域名解析到主机

---

参考

[说明 · acmesh-official/acme.sh Wiki](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)

[Use v2ray-plugin after Nginx · Issue #48 · shadowsocks/v2ray-plugin](https://github.com/shadowsocks/v2ray-plugin/issues/48)