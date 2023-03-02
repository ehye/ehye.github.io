---
title: 使用 branch 对 hexo 备份
date: 2017-09-12 20:41:13
categories: Note
tags: 
    - hexo
    - git
---
因为文章源文件都在本地 hexo 文件夹中，所以备份必不可少，而新建一个 repository 过于臃肿，因此利用 branch 进行备份<!--more-->

---

操作如下

1. 将网站 clone 到本地

    ```sh
    git clone git@github.com:username/username.github.io
    ```

2. 创建空的新分支

    ```sh
    git checkout --orphan <branchname>
    ```

3. 清空网站文件夹，把所需的 hexo 文件复制进来add

    ```sh
    git add .
    ```

4. 设置上游分支并推送

    ```sh
    git commit -am"initial commit"
    git push --set-upstream origin <branchname>
    ```

    到此，远程仓库就有两个 branch 了

5. 将 .git 文件夹复制到 Hexo 文件夹

6. 后续日常更新
只需在 Hexo 文件夹中进行`commit` `push` 操作
也可重新 clone 分支

    ```sh
    git clone -b <branchname> git@github.com:username/username.github.io
    ```

---
