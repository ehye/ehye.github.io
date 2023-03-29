---
title: 使用 Nginx 代理 OpenAI API
date: 2023-03-29 11:52:58
categories: Tips
tags:
    - Nginx
---


突破封锁，配置代理访问 OpenAI API <!-- more -->

---

使用以下配置，即可利用代理服务转发调用 OpenAI API

```nginx
location /api/v1/ {
    
    proxy_pass  https://api.openai.com/v1/;

    proxy_http_version 1.1;
    chunked_transfer_encoding on;
    proxy_max_temp_file_size 0;
    proxy_buffering off;
    proxy_cache off;

    proxy_set_header Connection '';
    proxy_set_header Host api.openai.com;
    proxy_set_header content-type "application/json";
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    proxy_connect_timeout              15s;
    proxy_send_timeout                 15s;
    proxy_read_timeout                 15s;
}
```

{% note default %} 

让 ChatGPT 来解释一些配置

> To enable streaming of the response, `chunked_transfer_encoding` is set to `on`. This allows the response to be sent to the client in a series of small chunks rather than all at once.

> To ensure that the response is not buffered, `proxy_buffering` is set to `off`. This allows the response to be sent to the client as soon as it is received from the upstream server.

> Finally, `proxy_max_temp_file_size` is set to `0` to ensure that no temporary files are created when buffering the response.

{% endnote %}

配置保存后 `sudo service nginx restart` 重启服务并验证 

现在就可以使用 `https://<your_domain>/api/v1/` 来代理访问 API OpenAI API

例如用来调用 ChatGPT API

```curl
curl --location 'https://<your_domain>/api/v1/chat/completions' \
--header 'Authorization: Bearer sk-xxxxxxxx' \
--header 'Content-Type: application/json' \
--data '{
  "model": "gpt-3.5-turbo",
  "messages": [
    {
      "role": "user",
      "content": "hello"
    }
  ]
}'
```

---

