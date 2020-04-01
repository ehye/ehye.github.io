---
title: 使用 Travis 将 hexo 博客自动部署到 Github Pages
date: 2020-03-28 18:39:19
categories: Tips
tags:
    - Travis
    - hexo
    - CI/CD
---

只需要考虑写作本身，将部署的事情交给程序处理吧<!-- more -->

到 Github 应用市场[安装 Travis](https://github.com/marketplace/travis-ci)

到 [Developer settings](https://github.com/settings/tokens) 生成 `Personal access tokens`，要勾选`repo`这整个分支

记下生成的密钥，到 Pages 的 Travis 设置页面找到 `Environment Variables`，新增一个 name 为 `GH_TOKEN`， value 为这个密钥的值

在你的 hexo 跟目录下创建 `.travis.yml`

```yml
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - hexo
install:
  - npm install -g hexo
  - npm install
before_script:
  - mkdir themes/next
  # clone next theme
  - curl -s https://api.github.com/repos/theme-next/hexo-theme-next/releases/latest | grep tarball_url | cut -d '"' -f 4 | wget -i - -O- | tar -zx -C themes/next --strip-components=1
  # copy the custom next theme config file
  - no | mv themes/next/* -i themes/hexo-theme-next/
script:
  - hexo clean
  - hexo generate
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: hexo
  target_branch: master
  local-dir: public
```

因为 next 主题我是单独上传一个 `_config.yml` 主题配置文件，所以多了一个 `before_script` 阶段，用于替换自己配置文件

{% note default %}
默认的 `provider: pages` 会将生成的html推送到 gh-pages 分支
但是 Github Pages 已不可绑定除了默认的 master 以外的分支
因此要指定手动指定 `target_branch: master`
{% endnote %}

从此，写作完成执行 `git commit` 和 `git push` 后，Travis 就会自动构建并部署到 Github Pages 上了，免去了配置 hexo 就算在另一台电脑上也可以轻松发布

---
