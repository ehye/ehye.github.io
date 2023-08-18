---
title: OpenWrt折腾记录06
date: 2023-08-02 14:56:56
categories: Router
tags:
    - OpenWrt
    - AdGuardHome
    - DNSMasq
---

安装 AdGuardHome 并配置为DNS服务器<!-- more -->

<img src="https://cdn.adguard.info/website/adguard.com/products/home/home.svg" class="full-image" />

# 安装

```bash
opkg update
opkg install adguardhome
```

设置为开机启动并启动服务

```bash
service adguardhome enable
service adguardhome start
```

# 配置

1. 打开 http://192.168.1.1:3000/ （192.168.1.1为路由器地址）
1. 设置管理页面在`3001`端口
1. 设置监听`53`端口

# 设置DNS服务器

## DNSMasq

为了让 AGH 能够监听53端口，将 DNSMasq 监听端口改为54

```bash
# Get the first IPv4 and IPv6 Address of router and store them in following variables for use during the script.
NET_ADDR=$(/sbin/ip -o -4 addr list br-lan | awk 'NR==1{ split($4, ip_addr, "/"); print ip_addr[1] }')
echo "Router IPv4 : ""${NET_ADDR}"

# 1. Enable dnsmasq to do PTR requests.
# 2. Reduce dnsmasq cache size as it will only provide PTR/rDNS info.
# 3. Disable rebind protection. 
# 4. Move dnsmasq to port 54.
uci set dhcp.@dnsmasq[0].noresolv='0'
uci set dhcp.@dnsmasq[0].cachesize='1000'
uci set dhcp.@dnsmasq[0].rebind_protection='0'
uci set dhcp.@dnsmasq[0].port='54'

uci -q delete dhcp.@dnsmasq[0].server
uci add_list dhcp.@dnsmasq[0].server="${NET_ADDR}"
```

设置 DHCP 通告网关和 DNS 为路由器IP

```bash
#Delete existing config ready to install new options.
uci -q delete dhcp.lan.dhcp_option
uci -q delete dhcp.lan.dns

# DHCP option 6: which DNS (Domain Name Server) to include in the IP configuration for name resolution
uci add_list dhcp.lan.dhcp_option='6,'"${NET_ADDR}"

#DHCP option 3: default router or last resort gateway for this interface
uci add_list dhcp.lan.dhcp_option='3,'"${NET_ADDR}"

# Save changes
uci commit dhcp

# Restart dnsmasq service to reflect changes
/etc/init.d/dnsmasq restart
```

确认启用 SLAAC 和 DHCPv6

```bash /etc/config/dhcp
...
config dhcp 'lan'
        option dhcpv4 'server'
        option dhcpv6 'server'
        option ra 'server'
        list ra_flags 'managed-config'
        list ra_flags 'other-config'
...
```

## AdGuardHome

1. 获取路由器本地 ip

    ```bash
    ip -o -4 addr list br-lan | awk 'NR==1{ split($4, ip_addr, "/"); print ip_addr[1] }'
    ```

1. 编辑 `/etc/adguardhome.yaml`

    - 绑定DNS服务地址

        ```yaml
        dns:
          bind_hosts:
            - 127.0.0.1
            - 192.168.1.1 # inet
            - ::1
          port: 53
        ```

    - 配置上游DNS为谷歌DNS

        ```yaml
        upstream_dns:
            - 8.8.8.8
            - 8.8.4.4
            - 2001:4860:4860::8888
            - 2001:4860:4860::8844
        ```

    - 限制日志大小
        
        ```yaml
        querylog:
            interval: 24h
            size_memory: 1000
            enabled: true
            file_enabled: true
        log:
            file: ""
            max_backups: 0
            max_size: 10
            max_age: 3
            compress: false
            local_time: false
            verbose: false
        ```

1. 检查配置是否有错误

    ```bash
    /usr/bin/AdGuardHome -c /etc/adguardhome.yaml --check-config
    ```

    确保 `/etc/resolv.conf` 使用本地解析

    ```bash
    root@OpenWrt:~# cat /etc/resolv.conf
    search lan
    nameserver 127.0.0.1
    nameserver ::1
    ```

1. 重启服务

    ```bash
    /etc/init.d/adguardhome restart
    ```

1. 检查DNS服务是否有效

    ```bash
    root@OpenWrt:~# nslookup google.com
    Server:         127.0.0.1
    Address:        127.0.0.1:53

    Non-authoritative answer:
    Name:   google.com
    Address: 2404:6800:4012:1::200e

    Non-authoritative answer:
    Name:   google.com
    Address: 172.217.160.110

    root@OpenWrt:~# nslookup google.com ::1
    Server:         ::1
    Address:        [::1]:53

    Non-authoritative answer:
    Name:   google.com
    Address: 2404:6800:4012:1::200e

    Non-authoritative answer:
    Name:   google.com
    Address: 172.217.160.110

    root@OpenWrt:~# nslookup google.com 192.168.1.1
    Server:         192.168.1.1
    Address:        192.168.1.1:53

    Non-authoritative answer:
    Name:   google.com
    Address: 2404:6800:4012:1::200e

    Non-authoritative answer:
    Name:   google.com
    Address: 172.217.160.110
    ```

---

[OpenWrt AdGuard Home 101 ( DNSMASQ ) - OpenWrt Forum](https://openwrt.org/docs/guide-user/services/dns/adguard-home)
[[OpenWrt Wiki] AdGuard Home](https://forum.openwrt.org/t/openwrt-adguard-home-101-dnsmasq/110864)