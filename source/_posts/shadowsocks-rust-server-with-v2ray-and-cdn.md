---
title: Setting up a Shadowsocks-rust server with v2ray plugin and CDN
date: 2022-10-30 22:51:08
categories: Note
tags:
  - Oracle cloud
  - shadowsocks-rust
  - V2ray
---

One Oracle Cloud account could create two free instance with mutable IP address.
<!-- more -->

---

## Conver private key form oracle cloud

Use PuTTYgen load the private key and save the private key as putty's format

1. Open **Console connection**
   
   {% asset_img ssh0.png %}

1. Download private key file from panel

   {% asset_img ssh1.png %}
   
1. Open PuTTYgen.

1. Click **Load**, and select the private key file which the extension is `.key`.

1. Click **Save private key**

---

## Install snap

```bash
sudo apt update
sudo apt install snapd
```

## Install shadowsocks-rust

```bash
sudo snap install shadowsocks-rust
```

## Install V2ray plugin

```bash
tar -xzvf v2ray-plugin-linux-amd64-v1.2.0.tar.gz
sudo cp v2ray-plugin_linux_386 /var/snap/shadowsocks-rust/common/v2ray-plugin
```

## Shadowsocks Configuration

The configuration file of shadowsocks-rust is `/var/snap/shadowsocks-rust/common/etc/shadowsocks-rust/config.json`

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

## Open firewall

```bash
sudo firewall-cmd  --permanent --zone=public --add-port=8081/tcp
sudo firewall-cmd  --reload
```

## Nginx Configuration

```bash
sudo vim /etc/nginx/sites-enabled/default
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

Use `ngingx -t` to check

## Start the daemon

```bash
sudo snap start --enable shadowsocks-rust.ssserver-daemon
```

## View the the log

```bash
sudo snap logs shadowsocks-rust.ssserver-daemon
```

## Cloudflare Configuration

Login into Cloudflare dashboard

Find the SSL/TLS -> Overview, Set the encryption mode to **Full**

{% asset_img cdn0.png %}

{% note warning %}
If your Nginx is configured to redirect HTTP request to HTTPS, and has self signed certificate on the server like generate by [acme.sh](https://github.com/acmesh-official/acme.sh), then set the encryption mode to **FULL**, otherwise you will get `TOO MANY REDIERCTS`.
{% endnote %}


You can check the CDN whether it is taking effects by using tool like `ping` or `nslookup`. 

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

**Then you are connected to the Internet !**

---

[Opening port 80 on Oracle Cloud Infrastructure Compute node](https://stackoverflow.com/a/54835902/6575354)

[shadowsocks-libev snap can't find custom downloaded plugin](https://github.com/shadowsocks/shadowsocks-libev/issues/2633#issuecomment-589652864)

[Why do I have to change from Flexible to Full to solve my "too many redirects" problem with cloudflare?](https://stackoverflow.com/q/70851543)
