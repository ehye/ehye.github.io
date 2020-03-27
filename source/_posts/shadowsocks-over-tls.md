---
title: Shadowsocks over websocket (HTTPS) 配置记录
date: 2020-03-27 18:53:20
categories: Tips
tags:
    - Nginx
    - Shadowsocks
    - TLS
    - V2Ray
---

裸连 Shadowsocks 在特殊时期会被无差别攻击，只有伪装成 HTTPS 才能续命
<!-- more -->

---
借助 Nginx 并配合 V2Ray 插件，根据路由判断是否为梯子的数据，将其转发到梯子程序，则可实现 HTTPS 与梯子共用443端口

## 原料

- shadowsocks-libev
- Nginx
- TLS 证书 [acme.sh](https://github.com/acmesh-official/acme.sh)
- v2ray-plugin [release](https://github.com/shadowsocks/v2ray-plugin/releases)

## 配置 Nginx 及证书

1. 颁发 TLS 证书

    ```bash
    acme.sh  --issue -d mydomain.com   --standalone
    ```

2. 复制/安装

    不要直接使用`~/.acme.sh/`下的文件，可以复制到`/etc/nginx/certs/`下，然后使用脚本安装

    ```bash
    acme.sh --installcert -d mydomain.com \
    --key-file       /etc/nginx/certs/mydomain.com.key  \
    --fullchain-file /etc/nginx/certs/fullchain.cer \
    --reloadcmd     "service nginx force-reload"
    ```

    配置好的 Nginx 文件如下

    ```nginx
    server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;
        ssl_certificate /etc/nginx/certs/fullchain.cer;
        ssl_certificate_key /etc/nginx/certs/mydomain.com.key;
        ssl_session_timeout 3m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;

        server_name mydomain.com;

        location /ray {
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
    acme.sh  --upgrade  --auto-upgrade
    ```

## Shadowsocks 配置

- 服务端配置

    `/etc/shadowsocks-libev/config.json`

    ```json
    {
        "server":"localhost",   # 只允许从本地访问
        "mode":"tcp_only",
        "server_port":8008,     # 同Nginx配置文件
        "local_port":1080,
        "password":"***",
        "timeout":5,
        "method":"chacha20-ietf-poly1305",
        "plugin":"v2ray-plugin",
        "plugin_opts":"server;host=mydomain.com;path=/ray"
    }
    ```

- Windows 客户端配置（部分）

    `gui-config.json`

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

---

参考

[Use v2ray-plugin after Nginx · Issue #48 · shadowsocks/v2ray-plugin](https://github.com/shadowsocks/v2ray-plugin/issues/48)
