---
title: 常用软件代理设置
date: 2023-03-13 09:24:27
categroies: Note
tags:
    - proxy
---

记录一些常用软件的代理设置方式<!-- more -->

## Git

- 使用 Git Bash

```bash
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080

git config --global --unset http.proxy
git config --global --unset https.proxy
```

- 编辑 .gitconfig

```bash .gitconfig
[https]
    proxy = https://127.0.0.1:1080
[http]
    proxy = http://127.0.0.1:1080
```

## yarn

```bash
yarn config set proxy http://127.0.0.1:1080
yarn config set https-proxy http://127.0.0.1:1080

yarn config delete proxy
yarn config delete https-proxy

yarn config get registry
yarn config set registry https://registry.npmmirror.com
yarn config set registry https://registry.yarnpkg.com
yarn config delete registry
```

## npm

- 使用命令

```bash
npm config set proxy http://127.0.0.1:1080
npm config set https-proxy http://127.0.0.1:1080

npm config delete proxy
npm config delete https-proxy

npm config get registry
npm config set registry https://registry.npmmirror.com
npm config set registry https://registry.npmjs.org
npm config delete registry
```

- 编辑 .npmrc

```cmd .npmrc
proxy=http://127.0.0.1:1080
https-prox=https://127.0.0.1:1080
```

## WSL2

添加到 ~/.bashrc 末尾

```bash .bashrc
host_ip=$(cat /etc/resolv.conf | grep "nameserver" | cut -f 2 -d " ")
export http_proxy="http://$host_ip:1080"
export https_proxy="http://$host_ip:1080"
```

对于 Shadowsocks, 勾选 `Allow other Devices to connect`

### Get Host IP

```bash
ip route | awk 'NR==1 {print $3}'
```

---
