---
title: Git Cheatsheets
categorties: Note
tags:
    - git
---

## config

- Create a new repository

    ```bash
    echo "" >> README.md
    git init
    git add README.md
    git commit -m "first commit"
    git remote add origin https://github.com/[username]/[repository].git
    git push -u origin master
    ```

- Push an existing repository

    ```bash
    git remote add origin https://github.com/<username>/<repository>.git
    git push
    ```

- Push to a licence added repository

    ```bash
    git pull --allow-unrelated-histories https://<reop>.git
    git push --set-upstream origin master
    ```

- 修改当前的 repo 的用户名

    ```bash
    git config user.name <username>
    ```

- 修改当前的 repo 的邮箱

    ```bash
    git config user.email <email>
    ```

- 全局用户名邮箱

    ```bash
    git config --global user.name <username>
    git config --global user.email <email>
    ```

- 重装后加 key

    ```bash
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/id_rsa
    ```

    > [Adding your SSH key to the ssh-agent](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)

- 配置代理

    ```conf
    [http]
        proxy = socks5://127.0.0.1:7070
    ```

- 修改仓库地址

    查看

    ```bash
    git remote -v
    ```

    改成用 SSH 免输密码

    ```bash
    git remote set-url origin git@github.com:ehye/ehye.github.io.git
    ```

## pull

- 拉取单个文件夹

    1. cd into the top of your repo copy
    2. `git fetch`
    3. `git checkout HEAD path/to/your/dir/or/file`

- Where "`path/...`" in (3) starts at the directory just below the repo root containing your "`.../file`"

- NOTE that instead of "HEAD", the hash code of a specific commit may be used, and then you will get the revision (file) or revisions (dir) specific to that commit.

    > [How to pull specific directory with git](https://stackoverflow.com/a/4048993/)

## commit

- 修改上一次 commit

    ```bash
    git commit -a --amend -m "my message here"
    ```

- 忽略文件（本地）

    ```bash
    git update-index --assume-unchanged PATH
    ```

- 取消忽略（本地）

    ```bash
    git update-index --no-assume-unchanged –path
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

## bracnh

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
