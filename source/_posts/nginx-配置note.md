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
