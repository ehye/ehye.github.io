---
title: 在多个仓库部署同一个 hexo 博客
date: 2018-08-01 12:39:36
categories: Blog
tags:
	- git
	- hexo
---

GitHub Pages 在国内访问会不可避免的慢，因此可以使用国内类似的平台提供的服务，来加快从国内访问的速度<!--more-->

---

# Hexo 设置

在 `_config.yml` 中修改参数，添加仓库地址

```
deploy:
- type: git
  repo:
- type: git
  repo:
```

# git 设置

打开 Hexo 生成后的 `public\.git\config` 文件
添加一个叫做 all 的 remote，将每个仓库的地址添加到下方，并保持 hexo 备份分支不变

```
[remote "all"]
	url = git@github.com:{user_name}/{user_name}.github.io.git
	url = git@git.coding.net:{user_name}/{user_name}.coding.me.git
[branch "hexo"]
	remote = all
	merge = refs/heads/hexo
```

然后在第二仓库添加你使用的 SSH KEY ，具体查看各平台的帮助文档

# 推送

设置上游分支并推送，这里使用之前定义的名为`all`的 remote 配置，后面跟随分支名`hexo`
```
git push --set-upstream all hexo
```

到此，每个远程仓库就有两个 branch 了

后续只要正常地使用`hexo d`和`git push`即可完成博客部署以及备份分支的推送

---
**ref**
> https://stackoverflow.com/questions/4255865/git-push-to-multiple-repositories-simultaneously