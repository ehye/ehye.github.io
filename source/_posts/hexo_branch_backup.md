---
title: 使用 branch 对 hexo 备份
date: 2017-09-12 20:41:13
categories: web
tags: 
	- hexo
	- blog
	- git
---
因为文章源文件都在本地 hexo 文件夹中，所以备份必不可少
而新建一个 repository 过于臃肿，因此利用 branch 进行备份

操作如下

1. 将网站 clone 到本地

2. 创建空的新分支
```
git checkout --orphan <branchname>
```

3. 清空网站文件夹，把所需的 hexo 文件复制进来

4. 设置上游分支并推送
```
git push --set-upstream origin hexo
```

5. 后续日常更新
```
git clone -b <branchname> git@github.com:username/username.github.io
```
复制更换文件后 commit push

缺点是每次备份都要先 clone 一次站点文件