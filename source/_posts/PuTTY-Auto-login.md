---
title: PuTTY 自动登录设置
date: 2018-01-19 00:18:32
categories: Web 
tags:
	- SSH
	- PuTTY
---

1. 打开 PuTTYgen，点击 generate ，不断在进度条下空白区域移动鼠标，直到生成密匙。

2. 点击 Save private key 将私匙保存起来

3. 登陆到远程节点，在~（home/username）路径下创建".ssh"文件夹，确保这个文件夹只有自己拥有操作权限：
```bash
$ mkdir -m 700 .ssh
```
4. 进入文件夹，创建公匙文件"authorized_keys"，将PuTTYgen文本框中生成的公匙复制粘贴过来，保存：
```bash
$ cd .ssh
$ vi authorized_keys
```
5. 打开PuTTY，进入 Connection/SSH/Auth 选项卡，在 Private key file for authentication 一栏中填入私匙地址

6. Connection/Data 在 Auto-login username 填入要自动登入的用户名

7. 回到PuTTY的Session选项卡，将更改保存一下，搞定。

> http://sfau.lt/b5cQwW