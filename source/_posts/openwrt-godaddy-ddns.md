---
title: OpenWrt折腾记录02
date: 2019-01-31 17:05:58
categories: Router
tags:
	- OpenWrt
	- DDNS
---

配置动态域名解析，在外网通过域名来访问路由器<!-- more -->

# 软件

安装动态DNS客户端

```sh
opkg install luci-app-ddns luci-i18n-ddns-zh-cn
```

安装GoDaddy脚本

```sh
opkg install ddns-scripts_godaddy.com-v1
```

安装uHTTPd

```sh
opkg install luci-app-uhttpd luci-i18n-uhttpd-zh-cn
```

# 配置

1. DNS改用DNSPod（可选）

2. [生成API密钥](https://developer.godaddy.com/keys)
Environment选`Production`
记住Key和Secret

3. 配置DDNS客户端
`查询主机名`和`域名`填自己的域名
`用户名`填Key，`密码`填Secret
`CA 证书路径`填证书和密钥所在的文件夹就行
获取IPv6的方法：接口设置为br-lan

    ```sh
    option use_ipv6 '1'
    option interface 'lan'
    option ip_source 'interface'
    option ip_interface 'br-lan'
    ```

4. 配置HTTPS服务器
运营商一般是屏蔽80端口的，干脆就用HTTPS。修改`/etc/config/uhttpd`

    ```sh
    option cert '/etc/ssl/mycert.crt'
    option key '/etc/ssl/mycert.key'
    option rfc1918_filter '0'
    option redirect_https '1'
    ```

---

ref
https://openwrt.org/docs/guide-user/services/ddns/client
https://openwrt.org/docs/guide-user/base-system/ddns