---
title: 如何使用 Nginx 反向代理 ASF-ui
date: 2020-01-09 18:28:27
categories: Tips
tags:
    - Ubuntu
    - ArchiSteamFarm
---

如何在成功运行 ASF 的机子上用 Nginx 实现在指定路径访问 ASF-ui<!-- more -->

例如，想通过 `example.com/asf` 访问 ASF-ui，则修改 Nginx 配置如下

```nginx
location /asf/api/nlog {
    proxy_pass http://127.0.0.1:1242/api/nlog;
    # proxy_set_header Host 127.0.0.1; # Only if you need to override default host
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $host:$server_port;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Real-IP $remote_addr;
    # We add those 3 extra options for websockets proxying, see https://nginx.org/en/docs/http/websocket.html
    proxy_http_version 1.1;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Upgrade $http_upgrade;
}

location /asf/ {
    proxy_pass http://127.0.0.1:1242/;
    # proxy_set_header Host 127.0.0.1; # Only if you need to override default host
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $host:$server_port;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Real-IP $remote_addr;
}

```

---

参考

[Nginx初学者折腾asf时想实现一个功能](https://keylol.com/t551721-1-1)

[我可以将 ASF 的 IPC 部署在 Apache 或者 Nginx 的反向代理后吗？](https://github.com/JustArchiNET/ArchiSteamFarm/wiki/IPC-zh-CN#%E6%88%91%E5%8F%AF%E4%BB%A5%E5%B0%86-asf-%E7%9A%84-ipc-%E9%83%A8%E7%BD%B2%E5%9C%A8-apache-%E6%88%96%E8%80%85-nginx-%E7%9A%84%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E5%90%8E%E5%90%97)
