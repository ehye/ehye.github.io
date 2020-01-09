---
title: 用 Github Pages 与 hexo 搭建个人博客
date: 2016-12-17 00:06:33
categories: Blog
tags: 
	- hexo
	- blog
---
> Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

<!-- more -->

--------------------------
 
# 安装

安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：

- [Node.js](http://nodejs.org/)
- [Git](http://git-scm.com/)

## 淘宝 NPM 镜像

_免梯子，使用[淘宝npm镜像](https://npm.taobao.org/)安装npm工具_
_使用定制的 cnpm (gzip 压缩支持) 命令行工具代替默认的 npm:_

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

继续使用 cnpm 即可完成 Hexo 的安装

```batch
cnpm install -g hexo-cli
```

--------------------------

建站
===
安装 Hexo 完成后，执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件
```
$ hexo init <folder>
$ cd <folder>
$ cnpm install
```
> 如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。

## 测试 ##
一切成功的话，此时可以运行$ hexo s启动服务器
默认情况下，访问网址为：http://localhost:4000/ 

--------------------------

配置
===
## 部署到Github ##
需要先提前安装一个扩展：
```
$ npm install hexo-deployer-git --save
```
接着在配置文件_config.xml中作如下修改：
```
deploy:
  type: git
  repo: git@github.com:username/username.github.io.git
  branch: master
```
把 username 替换成你的Github用户名，然后在命令行中执行
```
hexo d
```
即可完成部署。
## Local Search
添加自定义站点内容搜索
安装 hexo-generator-searchdb，在站点的根目录下执行以下命令：
```
npm install hexo-generator-searchdb --save
```
编辑 站点配置文件，新增以下内容到任意位置：
```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```