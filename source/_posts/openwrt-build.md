---
title: OpenWrt折腾记录06
date: 2023-02-13 17:45:52
categories: Router
tags:
    - OpenWrt
    - WSL
---

使用 WSL 编译 Openwrt 固件及安装包<!-- more -->

---

# WSL 安装必要软件

以 Ubuntu 为例

```bash
sudo apt update
sudo apt install build-essential clang flex bison g++ gawk gcc-multilib gettext \
git libncurses5-dev libssl-dev python3-distutils rsync unzip zlib1g-dev \
file wget
```

# SDK

使用 SDK 编译目标平台所需要的 package

从 release 获取 SDK，要下载的文件名为 `openwrt-sdk-*.tar.xz`

解压到工作文件夹

`tar -xf openwrt-sdk-*.tar.xz -C <dir_path>/openwrt-sdk`

`cd openwrt-sdk`

## 更新 feeds

获取最新的 feeds: `./scripts/feeds update -a`

## 获取 package
 
使用脚本： `./scripts/feeds install <PACKAGENAME>`

使用 git： `git clone https://github.com/*/<repo_name>.git package/<repo_name>`

## 配置 package

`make menuconfig` 选择要编译的包，配置保存于 `.config`

{% note info %} 

当 package 没出现在列表时，尝试清除临时文件夹

```bash
cd ~/openwrt/
rm -rf tmp
make menuconfig
```

{% endnote %}


## 生成 package

`make package/shadowsocks-rust/compile V=99`

构建成功的 package 位于 `bin/`

# Image Builder

使用 Image Builder 来生成自己的可刷镜像文件

删除 WSL 中 Windows 的环境变量

```bash
sudo tee -a /etc/wsl.conf << EOF > /dev/null
[interop]
appendWindowsPath = false
EOF
exit
```

重启以应用

`wsl --shutdown`

验证 PATH 中不包含 Windows 的路径

`echo ${PATH}`

## 获取源码

```bash
tar -J -x -f openwrt-imagebuilder-*.tar.xz
cd openwrt-imagebuilder-*/
```

执行 `make info` 获取目标平台 profile

### 生成镜像

`make image PROFILE="profile-name" PACKAGES="pkg1"`

构建成功的镜像位于 `bin/`

---

**参考**

[Build system setup](https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem#build_system_setup)
[OpenWrt build system – Usage](https://oldwiki.archive.openwrt.org/doc/howto/build#updating_feeds)
[Using the Image Builder](https://openwrt.org/docs/guide-user/additional-software/imagebuilder)
