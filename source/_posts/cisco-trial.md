---
title: Cisco 网络实验室期末测试B组
date: 2017-06-18 19:28:30
categories: MOOC
tags: 
	- Cisco
---
本题所用的操作：路由、单臂路由、VLAN、ACL、NAT、无线网络等
<!--more-->
{% asset_img outline.png %}

# 实践内容：

- 路由和单臂路由配置，实现A、B、C部门计算机能访问Web、FTP服务
- Web、FTP服务器配置静态NAT，实现A、B、C部门计算机能访问Web、FTP服务
- 配置安全性，禁止B部门的计算机访问FTP服务器
- 配置安全性，禁止B部门“恶意入侵”者访问Web服务
- 管理员计算机能telnet登录R1路由器进行管理

# 路由和单臂路由配置

## 配置过程

S0
```
Switch(config)#vlan 5
Switch(config-vlan)#name vlan5
Switch(config)#interface FastEthernet0/1
Switch(config-if)#switchport access vlan 5
Switch(config)#interface FastEthernet0/2
Switch(config-if)#switchport access vlan 5
Switch(config)#interface FastEthernet0/24
Switch(config-if)#switchport access vlan 5
```
S1
```
Switch(config)#vlan 5
Switch(config-vlan)#name vlan5
Switch(config)#vlan 10
Switch(config-vlan)#name vlan10
Switch(config)#interface FastEthernet0/24
Switch(config-if)#switchport access vlan 5
Switch(config)#interface FastEthernet0/1
Switch(config-if)#switchport access vlan 10
Switch(config)#interface FastEthernet0/2
Switch(config-if)#switchport access vlan 10
Switch(config)#interface FastEthernet0/23
Switch(config-if)#switchport mode trunk
```
R1
```
Router(config)#int fa1/0.1
Router(config-subif)#en do 5
Router(config-subif)#ip add 192.168.222.1 255.255.255.0
Router(config-subif)#int fa1/0.2
Router(config-subif)#en do 10
Router(config-subif)#ip add 192.168.29.1 255.255.255.0
```

# 配置静态NAT

## 配置过程
---
R1
```
Router(config)#router ospf 1
Router(config-router)#netw 172.16.29.0 0.0.0.255 area 0
Router(config-router)#netw 220.220.29.0 0.0.0.255 area 0
Router(config-router)#netw 192.168.223.0 0.0.0.255 area 0
Router(config-router)#netw 192.168.222.0 0.0.0.255 area 0
Router(config-router)#netw 192.168.29.0 0.0.0.255 area 0
```
R2
```
Router(config)#router ospf 1
Router(config-router)#netw 220.220.29.0 0.0.0.255 area 0
Router(config-router)#netw 200.200.29.0 0.0.0.255 area 0
```
R3
```
Router(config)#ip nat in sour st 172.20.29.2 200.200.29.3
Router(config)#ip nat in sour st 172.21.29.2 200.200.29.4
Router(config-if)#ip nat in
Router(config-if)#int fa0/1
Router(config-if)#ip nat in
Router(config-if)#int se0/2/0
Router(config-if)#ip nat out
Router(config)#route ospf 1
Router(config-router)#netw 172.20.29.0 0.0.0.255 area 0
Router(config-router)#netw 172.21.29.0 0.0.0.255 area 0
Router(config-router)#netw 200.200.29.0 0.0.0.255 area 0
```

## 测试结果


{% asset_img web-ftp-pass1.png %}

{% asset_img web-ftp-pass2.png %}

{% asset_img web-ftp-pass3.png %}

# 禁止B部门访问FTP服务器

## 配置过程

R2
```
Router(config)#access-list 110 remark NO FTP
Router(config)#access-list 110 deny tcp 192.168.29.0 0.0.0.255 host 200.200.29.4 eq 21
Router(config)#access-list 110 deny tcp 192.168.29.0 0.0.0.255 host 200.200.29.4 eq 20
Router(config)#access-list 110 permit ip any any 
Router(config)#int se0/2/0
Router(config-if)#ip access-group 110 in
```

测试结果
--- 
{% asset_img deny-ftp.png %}

# 禁止“恶意入侵”者访问 Web

## 配置过程

R2
```
Router(config)#access-list 111 remark NO HTTP
Router(config)#access-list 111 deny tcp 192.168.29.3 0.0.0.0 200.200.29.3 0.0.0.0 eq  www
Router(config)#access-list 111 permit ip any any 
Router(config)#int se0/2/0
Router(config-if)#ip access-group 111 in
```

## 测试结果

{% asset_img deny-web.png %}

# telnet 登录R1路由器进行管理
## 配置过程

R2
```
Router#configure terminal 
Router(config)#line vty 0 4
Router(config-line)#password cisco
Router(config-line)#login
Router(config-line)#exit
Router(config)#enable password cisco
Router(config)#end
```

## 测试结果

{% asset_img telnet.png %}

# 实践总结

本次实践并不一帆风顺，在实际操作中遇到了一些问题：

- 不清楚在哪台路由器上配置 NAT ——其实只要把两台服务器映射给到公网就行了
- 单臂路由和 VLAN 配置语句不熟练——由于模拟器版本，一些封装语句和课本不一样
- ACL 添加出错——课本上的简写语句并不能直接使用，如 host 需写成 0.0.0.0

---
