---
title: PuTTY 自动登录设置
date: 2018-01-19 00:18:32
categories: Tips
tags:
	- SSH
	- PuTTY
---

1. 打开 PuTTYgen，点击 generate ，不断在进度条下空白区域移动鼠标，直到生成密匙。

2. 点击 Save private key 将私匙保存起来

3. 安装ssh
```bash
apt-get install ssh
```

4. 登陆到远程节点，在 `/home/<username>` 路径下创建 `.ssh` 文件夹，确保这个文件夹只有自己拥有操作权限：
```bash
$ mkdir -m 700 .ssh
```

5. 进入文件夹，创建文件"authorized_keys"，将PuTTYgen文本框中生成的公匙粘贴过来：（公钥以 ssh-rsa 开头并只有一行）
```bash
$ cd .ssh
$ vi authorized_keys
```

6. 关闭密码验证，打开 ssh 配置文件`vim /etc/config/dropbear`增加一行`option PasswordAuth 'off'`

7. 打开PuTTY，进入 Connection/SSH/Auth 选项卡，在 Private key file for authentication 中选择私匙文件

8. 定位到 Connection/Data 在 Auto-login username 填入要自动登入的用户名

9. 回到 Session 选项卡，将更改保存一下，搞定。

> http://sfau.lt/b5cQwW