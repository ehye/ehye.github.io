---
title: hexo 配置 firestore 记录阅读数
date: 2019-07-07 12:21:18
categories: Note
tags:
    - hexo
    - firebase
    - firestore
---

由于leancloud加强审查，遂弃用，改用firebase<!--more-->

---

# 安装

# 配置

[登录 Firebase](https://console.firebase.google.com/)
创建firestore
firestore在Database里

替换hexo过时的js引用
path to hexo\themes\hexo-theme-next\layout\_third-party\analytics\firestore.swig

```html
<script src="https://www.gstatic.com/firebasejs/6.2.4/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/5.10.1/firebase-firestore.js"></script>
```

# 疑问

本地登录作用是什么(`firebase login --interactive`)
登录重定向至的`localhost:9005`是什么


> https://firebase.google.com/docs/firestore/
> https://firebase.google.com/docs/web/setup#reserved-urls
> https://stackoverflow.com/questions/35368254/cannot-deploy-angular-app-on-firebase
