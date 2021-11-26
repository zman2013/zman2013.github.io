---
title: nginx-template
date: 2021-11-26 13:16:22
tags:
---

# 例子
1. 进入配置目录
```
cd /etc/nginx/sites-enabled
```
2. 新建文件 xiu66.site，内容如下:
```shell
server {
  listen 80;
  server_name xiu66.site;
  # client 可以携带的最大 body 为 20m
  client_max_body_size 20m;
  # 将请求都转发到 127.0.0.1:3000
  # 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求
  # 但是正则和最长字符串会优先匹配
  location / {
      proxy_pass_header Server;
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Scheme $scheme;
      proxy_pass http://127.0.0.1:3000;
  }
  # static 开头的请求会到 root 目录下寻找
  location /static/ {
    # 绝对路径，最终文件路径=root+${location}
    root /web/xiu66/dist/;
    index index.html;
  }
}
## 参考信息
1. [https://www.jianshu.com/p/38810b49bc29](https://www.jianshu.com/p/38810b49bc29)
2. [http://nginx.org/en/docs/http/ngx_http_core_module.html](http://nginx.org/en/docs/http/ngx_http_core_module.html)
```
# 配置 https
参考 [https://coolshell.cn/articles/18094.html](https://coolshell.cn/articles/18094.html)