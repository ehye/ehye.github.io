---
title: Shadowsocks over websocket (HTTPS) 配置记录
date: 2020-03-27 18:53:20
categories: Tips
tags:
    - Nginx
    - Shadowsocks
    - SSL
    - V2Ray
---

裸连 Shadowsocks 在特殊时期会被无差别攻击，只有伪装成 HTTPS 才能续命
<!-- more -->

---
借助 Nginx 并配合 V2Ray 插件，根据路由判断是否为梯子的数据，将其转发到梯子程序，则可实现 HTTPS 与梯子共用443端口

## 原料

- shadowsocks-libev
- Nginx
- SSL 证书 [acme.sh](https://github.com/acmesh-official/acme.sh) （DNSpod 等免费 SSL 证书亦可）
- DNS解析
- v2ray-plugin [release](https://github.com/shadowsocks/v2ray-plugin/releases)

## 配置 Nginx 及证书

1. 颁发 SSL 证书

    ```bash
    acme.sh --issue -d mydomain.com --standalone
    ```

2. 复制/安装

    不要让 Nginx 直接使用`~/.acme.sh/`下的文件，使用以下脚本可以复制到`/etc/nginx/certs/`下并安装

    ```bash
    acme.sh --installcert -d mydomain.com \
    --key-file       /etc/nginx/certs/mydomain.com.key  \
    --fullchain-file /etc/nginx/certs/fullchain.cer \
    --reloadcmd     "service nginx force-reload"
    ```

    安装并配置 Nginx

    ```nginx
    server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server; # 没有 ipv6 可以不写这行
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

    {% note info %}
    此例中 /ray 就是你要作为梯子访问的路径，以此达到网站和梯子共用443端口的目的
    {% endnote %}

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
        "mode":"tcp_only",
        "server_port":8008,
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
  - `server_port`要和Nginx配置文件中`proxy_pass`的端口一致
  - 因为是通过 Ngxin 转发，同时也配置了HTTPS，所以`plugin_opts`就不需要再使用证书了
  - path 要和在 Nginx 配置里 location 中规定的一致
    {% endnote %}

- Windows 客户端配置

    节选`gui-config.json`

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

[Use v2ray-plugin after Nginx · Issue #48 · shadowsocks/v2ray-plugin](https://github.com/shadowsocks/v2ray-plugin/issues/48)

[说明 · acmesh-official/acme.sh Wiki](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)
