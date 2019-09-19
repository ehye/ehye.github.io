---
title: git-cheatsheet
categorties: Note
tags:
	- git
---

# Note

- fetch is 下载

- pull is fetch and merge

- 合并 commit

# 常用命令

```bash
git commit -a --amend -m "my message here"
```

- 修改当前的project的用户名

```bash
git config user.name <用户名>
```

- 修改当前的project提交邮箱

```bash
git config user.email <邮箱>
```

- 全局用户名邮箱

```bash
git config  --global user.name <用户名>
git config  --global user.email <邮箱>
```

- 忽略文件（本地）

```bash
git update-index --assume-unchanged PATH
```

- 取消忽略

```bash
git update-index –no-assume-unchanged –path
```

- 所有被忽略的文件

```bash
git ls-files -v | grep '^h\ '
```

- 忽略文件/移除 add 操作（重置）

1. `git rm --cached <PATH>`
2. 更新 .gitignore
3. `git commit -m "don't track this"`

> [git忽略已经被提交的文件](https://segmentfault.com/q/1010000000430426)

- 移动分支指向

```bash
git rebase <des>
```

- 移动HEAD

```bash
git checkout <des>
```

- 切换分支

```bash
git checkout -b <branch>
```

- 重装后加key

```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa
```

> [Adding your SSH key to the ssh-agent](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)

- 配置代理

```REST
[http]
    proxy = socks5://127.0.0.1:7070
```
