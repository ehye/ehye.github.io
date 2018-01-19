---
title: 利用亚马逊云搭建 Shadowsock 服务器
date: 2017-03-09 16:23:12
categories: Web
tags:
	- Ubuntu
	- 梯子
	- AWS
	- Shadowsock
---

不耗一元电费，使用云主机搭建Shadowsock服务器<!--more-->


配置AWS
======

- 打开 [AWS官网](http://aws.amazon.com/cn/)
注册账号，需信用卡号码

- 注册完成后，在控制面板 选择EC2
{% asset_img 1.png %}

- 进入EC2控制台，右上角 推荐选择 亚太地区（东京）；EC2控制台刷新后，点击 启动实例
{% asset_img 2.png %}

- 选择 符合条件的免费套 Ubuntu Server 16.04 LTS (HVM), SSD Volume Type
{% asset_img 3.png %}

- 下个页面 默认的选择第一个即可， 点击 审核和启动
{% asset_img 4.png %}

- 下个页面 点击 启动
- 创建新密钥对，点击下载密钥对（建议备份），点击 启动实例
{% asset_img 5.png %}

- 这样就创建好一个服务器节点，点击查看实例

---

连通性设置
====
1. 选择相应实例的安全组
{% asset_img 8.png %}

2. 选择入站，编辑，添加允许所有流量
{% asset_img 8.png %}

3. 在本地用cmd 进行 ping 测试，通了就开始连接服务器

---

连接服务器
====

在实例上右键单击，选择连接，弹出教程，[使用 PuTTY 连接](https://docs.aws.amazon.com/zh_cn/console/ec2/instances/connect/putty)
{% asset_img 6.png %}
- [下载 PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) ， 选择64位版本


## 连接到 Linux 实例
### 转换私有密钥
1. 启动 **PuTTYgen** (例如，在开始菜单中，选择 All Programs > PuTTY > PuTTYgen)

2. 在菜单栏 **Key** 单击，选择 **SSH-2 RSA**
{% asset_img puttygen-key-type.png.png %}

3. 选择 **Load**，显示所有类型的文件，要找到 .pem 文件
{% asset_img puttygen-load-key.png %}

4. 选择您在启动实例时指定的密钥对的 .pem 文件，然后选择 打开。选择 OK 关闭确认对话框。

5. 选择 Save private key，以 PuTTY 可以使用的格式保存密钥。PuTTYgen 显示一条关于在没有口令的情况下保存密钥的警告。选择是
>Note
私有密钥的口令是一层额外保护，因此，即使您的私有密钥被泄露，在没有口令的情况下，该密钥仍不可用。使用口令的缺点是让自动化变得更难，因为登录到实例或复制文件到实例需要进行人为干预。

6. 为该密钥指定与密钥对相同的名称 (如，my-key-pair)。PuTTY 自动添加 .ppk 文件扩展名。

现在私有密钥格式是正确的 PuTTY 使用格式了，现在可以使用 PuTTY 的 SSH 客户端连接到实例

### 启动 PuTTY 会话

1. 启动 PuTTY

2. 在 Category 窗格中，选择 Session 并填写以下字段
 在 Host Name 框中，输入 **ubuntu**@*your_public_dns_name*
 在 Connection type 下，选择 SSH
 确保 Port 为 **22**
3. 在 Category 窗格中，展开 Connection，再展开 SSH，然后选择 Auth (身份验证) 
- 选择 Browse
- 选择您为密钥对生成的 .ppk 文件，然后选择打开
- (可选) 如果打算稍后重新启动此会话，则可以保存此会话信息以便日后使用。在类别树中选择会话，在 Saved Sessions 中输入会话名称，然后选择保存
- 选择打开以便开始 PuTTY 会话

---

使用一键配置脚本
===
> http://teddysun.com/342.html

```
wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

---

开始使用
===
在要上网的设备配置好Shadowsock客户端，输入上面的ss服务器信息，就可以魔法上网了