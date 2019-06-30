---
title: git-cheatsheet
categorties: Note
tags:
	- git
---

# fetch is 下载
# pull is fetch and merge

# 合并 commit
git commit -a --amend -m "my message here"

# 修改当前的project的用户名
git config user.name <用户名>

# 修改当前的project提交邮箱
git config user.email <邮箱>

# 全局用户名邮箱
git config  --global user.name <用户名>
git config  --global user.email <邮箱>

# 忽略文件（本地）
git update-index --assume-unchanged PATH

# 忽略文件/移除 add 操作（重置）
1. `git rm --cached PATH`
2. 更新 .gitignore
3. `git commit -m "don't track this"`

> https://segmentfault.com/q/1010000000430426

# 移动分支指向
git rebase <des>

# 移动HEAD
git checkout <des>

# 切换分支
git checkout -b <branch>