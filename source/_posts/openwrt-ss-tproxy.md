---
title: Openwrt折腾记录03
date: 2019-02-23 14:03:56
categories: Router
tags:
	- OpenWrt
	- shadowsocks
	- 透明代理
	- 科学上网
---

Openwrt+shadowsocks透明代理，实现路由器科学上网<!-- more -->

当局域网中设备较多时，对每个主机配置ss客户端将变得繁琐，此时最适合在路由器上实现科学上网

---

# 安装

一些东西在镜像源里是没有的

添加用于验证的 gpg key
```
wget http://openwrt-dist.sourceforge.net/packages/openwrt-dist.pub -O /tmp/openwrt-dist.pub
opkg-key add /tmp/openwrt-dist.pub
```
新增feeds
```
src/gz openwrt_dist http://openwrt-dist.sourceforge.net/packages/base/mipsel_24kc
src/gz openwrt_dist_luci http://openwrt-dist.sourceforge.net/packages/luci
```
更新并安装
```
opkg update
opkg install dns-forwarder
opkg install luci-app-dns-forwarder
opkg install shadowsocks-libev
opkg install luci-app-shadowsocks
```

这个ss客户端和镜像源里的ss不同


# 配置shadowsocks

## shadowsocks.json
通过luci配置或者自定义shadowsocks配置文件
```json
{
    "server": "*.*.*.*",
    "server_port": 1080,
    "password": "****",
    "method": "aes-128-cfb",
    "local_address": "0.0.0.0",
    "timeout": 5,
    "reuse_port": true
}
```

## 配置dnsmasq和ipset

将如下规则加入到防火墙自定义规则中，最后的1234是shadowsocks的透明代理端口

```bash
ipset -N gfwlist iphash
iptables -t nat -A PREROUTING -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1234
iptables -t nat -A OUTPUT -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1234
```

## 添加gfwlist配置文件

新建并进入目录`/etc/dnsmasq.d`，下载 [dnsmasq_gfwlist_ipset.conf](https://cokebar.github.io/gfwlist2dnsmasq/dnsmasq_gfwlist_ipset.conf)
*在这个列表就是墙列表，里面的域名将走127.0.0.1:5353，稍后对这个端口配置转发，转发到谷歌DNS进行解析*

## 修改dnsmasq配置

修改 `/etc/dnsmasq.conf`，在最后加入 `conf-dir=/etc/dnsmasq.d`

# 配置DNS

## 防污染
转发墙外域名到谷歌DNS进行解析

{% asset_img dns-forward.png %}

> 5353是多播DNS(mDNS)的端口

最后需要在自定义防火墙规则里再增加一句，来确保8.8.8.8是走代理的：
```
ipset add gfwlist 8.8.8.8
```

## 自定义DNS
对WAN口进行如图配置，将DNS手动指定为127.0.0.1，将域名解析交给dnsmasq

{% asset_img custom-dns.png %}

最后在DHCP/DNS设置中将你的ISP的DNS填入（或者使用公共DNS）,让国内域名走国内解析

{% asset_img dnsmasq-dns-forward.png %}

# 测试
http://www.ip111.cn/

---

ref

> http://openwrt-dist.sourceforge.net/
> https://github.com/aa65535/openwrt-dist-luci
> https://cokebar.info/archives/962
> https://ssr.tools/473