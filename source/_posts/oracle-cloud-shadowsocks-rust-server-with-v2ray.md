---
title: Setting up a Shadowsocks-rust server with v2ray plugin and Cloudflare CDN on Oracle cloud
date: 2022-10-30 22:51:08
categories: Tips
tags: 
    - Oracle cloud
    - shadowsocks-rust
    - v2ray
---

Every free account could create two instance with mutable IP address
<!-- more -->

---

## Conver private key form oracle cloud

Download private key file from panel

Use PuTTYgen load the private key and save the private key as putty's format

---

## Install snap

```bash
sudo apt update
sudo apt install snapd
```

## Install shadowsocks-rust

```bash
snap install shadowsocks-rust
```

## Open firewall

```bash
sudo firewall-cmd  --permanent --zone=public --add-port=8081/tcp
sudo firewall-cmd  --reload
```

## Nginx Configuration

```bash
vim /etc/nginx/sites-enabled/default
```

```nginx
location  /ladder {
    proxy_redirect              off;
    proxy_pass                  http://127.0.0.1:8081;
    proxy_http_version          1.1;
    proxy_set_header Host       $http_host;
    proxy_set_header Upgrade    $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

## Install V2ray plugin

```bash
tar -xzvf v2ray-plugin-linux-amd64-v1.2.0.tar.gz
sudo cp v2ray-plugin_linux_386 /var/snap/shadowsocks-rust/common/v2ray-plugin
```

## Shadowsocks Configuration

The path of config.json for shadowsocks-rust is `/var/snap/shadowsocks-rust/common/etc/shadowsocks-rust/`

```bash
sudo vim /var/snap/shadowsocks-rust/common/etc/shadowsocks-rust/config.json
```

```json
{
  "server": "localhost",
  "server_port": 8081,
  "method": "chacha20-ietf-poly1305",
  "password": "********",
  "mode": "tcp_only",
  "fast_open": false,
  "timeout": 5,
  "plugin": "v2ray-plugin",
  "plugin_opts": "server;path=/ladder;"
}
```

## Start the daemon

```bash
sudo snap start --enable shadowsocks-rust.ssserver-daemon
sudo snap logs shadowsocks-rust.ssserver-daemon
```

## Cloudflare Configuration

SSL/TLS -> Overview -> Full


## Client Configuration

```json
{
    "server": "mydomain.com",
    "server_port": 443,
    "password": "***",
    "method": "chacha20-ietf-poly1305",
    "plugin": "v2ray-plugin",
    "plugin_opts": "tls;host=mydomain.com;path=/ray",
    "timeout": 5
}
```

---


> [shadowsocks-libev snap can't find custom downloaded plugin](https://github.com/shadowsocks/shadowsocks-libev/issues/2633#issuecomment-589652864)
https://stackoverflow.com/questions/70851543/why-do-i-have-to-change-from-flexible-to-full-to-solve-my-too-many-redirects-p

> [Opening port 80 on Oracle Cloud Infrastructure Compute node](https://stackoverflow.com/a/54835902/6575354)