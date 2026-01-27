---
title: nginx-配置note
date: 2026-01-23 15:14:57
categories:[Frontend]
---

## nginx的作用
Nginx 是一个高性能的 Web 服务器 + 反向代理 + 负载均衡器。

### web服务器
Nginx 可以作为静态文件服务器，处理客户端的 HTTP 请求，返回 HTML、CSS、JavaScript 和图片等静态资源。
```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        root /var/www/html;
        index index.html index.htm;
    }
}
```
浏览器访问 example.com，Nginx 直接把文件丢回去.

### 反向代理
Nginx 可以作为反向代理服务器，接收客户端请求并将其转发到后端应用服务器（如 Node.js、Python、Java 等），然后将后端服务器的响应返回给客户端。

后端服务不直接暴露在公网。
```nginx
server {
    listen 80;
    server_name example.com;
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
### 负载均衡
Nginx 可以将客户端请求分发到多个后端服务器，以实现负载均衡，提高系统的可用性和性能。

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}
server { 
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
``` 

**负载均衡策略**
- 轮询（默认）：依次将请求分发到每个服务器。
- 权重：根据服务器的权重分配请求，权重高的服务器获得更多请求。
- IP 哈希：根据客户端的 IP 地址分配请求，确保同一客户端的请求总是分发到同一台服务器。

```nginx
ip hash;
server 127.0.0.1:8080 weight=3;
server 127.0.0.1:8081 weight=1;
```

> HTTP 请求：状态不能放在实例里，必须外置（Redis / DB），这样双活才成立。

> 长连接：连接一旦建立，就天然固定在某一个 Pod / 实例上，不需要额外处理“双活路由”。

#### 静态资源与动态请求分离
```nginx

server {
    listen 80;

    location /static/ { // 静态资源不占用后端线程
        root /data/web;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

#### HTTPS / SSL 终结（证书统一）
```nginx
server {
    listen 443 ssl;
    ssl_certificate     cert.pem;
    ssl_certificate_key key.pem;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```
后端不用管 HTTPS，全部交给 Nginx

## nginx的变量

### 内置变量
随时都能用的，来自 Nginx 核心请求上下文。
- $remote_addr: 客户端 IP 地址。
- $host: 请求的主机名。
- $request_uri: 包含参数的完整请求 URI。
- $uri: 不包含参数的请求 URI。
- $args: 请求参数部分。
- $http_user_agent: 客户端的 User-Agent 头信息。
- $http_referer: 请求的来源页面 URL。
- $server_port: 服务器监听的端口号。
- $scheme: 请求使用的协议（http 或 https）。
### 自定义变量
可以在配置文件中使用 `set` 指令定义自己的变量。
```nginx
set $my_var "some_value";
```
### header派生变量
可以通过 `$http_` 前缀访问请求头信息，将请求头名称中的连字符 `-` 替换为下划线 `_`，全部小写。

```http
X-User-Id: 123
```
可以通过 `$http_x_user_id` 访问其值 `123`。

### map生成的变量
```nginx
map $http_user_id $backend {
    default app_b;
    1001    app_a;
    1002    app_c;
}
```
`$backend` 变量将自动生成，并包含 `app_b`、`app_a` 和 `app_c` 三个值。

使用`proxy_pass http://$backend;`实现基于用户 ID 的请求路由。

## nginx配置

### 基本结构

```
main（全局
└── events
└── http
    ├── upstream
    ├── server
        └── location
```
### 示例配置文件

```nginx
worker_processes auto; // 默认为CPU核数

events {
    worker_connections 1024; // 一个 worker 能接多少连接
}

http {
    include mime.types;
    default_type application/octet-stream;

    sendfile on;
    keepalive_timeout 65;

    upstream digitalhuman { // 后端服务定义
        server 1.2.3.4:8000;
        keepalive 32;
    }

    server {
        listen 80; // 监听端口
        server_name _;

        location /digitalhuman_app/ { // 前端请求路径，正则匹配
            client_max_body_size 50m;

            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_buffering off;
            proxy_read_timeout 300s;

            proxy_pass http://digitalhuman/; // 这里加 / 表示去掉前缀
        }
    }
}
```

参考问答：
[nginx能做什么好玩的事情](https://www.zhihu.com/question/21483073)