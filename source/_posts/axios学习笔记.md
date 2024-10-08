---
title: Axios学习笔记
date: 2024-10-08 14:57:00
categories: [frontend]
---

## 前言
在Java开发实训项目中，使用到了Axios来发送请求，所以学习了Axios的使用。

教程参考：[Axios教程](https://www.axios-http.cn/docs/intro)

## Axios
Axios 是一个基于 promise 的 HTTP 库，可以用在浏览器和 node.js 中。

**promise是什么？**(简要理解，详细理解放到后面)

> Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了 Promise 对象。

### 安装
```bash
npm install axios
```
### 使用
这里还是以项目中的实际使用为例。

首先看一个完整的HTTP请求示例：
```javascript
POST /login HTTP/1.1   // 请求行
Host: example.com    // 请求头
Content-Type: application/json // 指明请求体的格式
Authorization: Bearer <token> // 指明身份验证信息

{  // 请求体
  "username": "john_doe",
  "password": "secret123"
}
```

在项目中，需要发送请求到后端，所以需要配置一个axios实例，这样可以统一配置请求头等信息。[请求配置参考](https://www.axios-http.cn/docs/req_config)

```javascript
import axios from 'axios'

const API_BASE_URL = 'http://localhost:8080/api'

// 自定义实例的默认配置
const instance = axios.create({
    baseURL: API_BASE_URL,
    headers: {
        'Content-Type': 'application/json'
    }
});
```

### 参数传递位置

**Query参数**：在URL中传递参数，如`/project?userId=1`，后端使用`@RequestParam`接收

- 位置：参数作为查询字符串附加在 URL 之后，例如：/project?userId=12345。
- 用途：通常用于 GET 请求，适合传递简单的键值对参数。

**路径参数**：在URL中传递参数，如`/project/1`, 后端使用`@PathVariable`接收

- 位置：参数作为 URL 路径的一部分，例如：/project/12345。
- 用途：通常用于 RESTful API 中的资源标识，适合传递资源 ID 等信息

**请求体参数**：在请求体中传递参数，如`{userId: 1}`, 后端使用`@RequestBody`接收
- 位置：参数作为请求体的一部分，例如：{userId: 12345}。

**示例：**

```javascript
// 使用 Query 参数
export const getProjectsByUserId = (userId) => {
    return instance.get(`/project`, { 
        params: {
            userId: userId
        }
    });
}

// 使用路径参数
export const deleteProjectById = (projectId) => {
    return instance.delete(`/project/${projectId}`);
}

// 使用请求体参数
export const addProject = (project) => {
    return instance.post(`/project/add`, project);
}

```

### 拦截器
拦截器可以在请求或响应被 then 或 catch 处理前拦截它们。

在项目中使用到了请求拦截器，每次请求时自动将 token 添加到请求头中。

```javascript
instance.interceptors.request.use(
    config => {
        const token = localStorage.getItem('token');
        if (token) {
            config.headers['Authorization'] = `Bearer ${token}`;
        }
        return config;
    },
    error => {
        return Promise.reject(error);
    }
);
```

响应拦截器：
    
```javascript
    // 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    return response;
  }, function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    return Promise.reject(error);
  });
```

### 响应结构
Axios返回的是一个Promise对象，所以可以使用`then`和`catch`方法处理响应。

一个响应结构应该包含如下信息：

```javascript
{
  // `data` 由服务器提供的响应
  data: {},

  // `status` 来自服务器响应的 HTTP 状态码
  status: 200,

  // `statusText` 来自服务器响应的 HTTP 状态信息
  statusText: 'OK',

  // `headers` 是服务器响应头
  // 所有的 header 名称都是小写，而且可以使用方括号语法访问
  // 例如: `response.headers['content-type']`
  headers: {},

  // `config` 是 `axios` 请求的配置信息
  config: {},

  // `request` 是生成此响应的请求
  // 在node.js中它是最后一个ClientRequest实例 (in redirects)，
  // 在浏览器中则是 XMLHttpRequest 实例
  request: {}
}
```

使用`.then()`方法处理响应：

```javascript
axios.get('/user/12345')
  .then(function (response) {
    console.log(response.data);
    console.log(response.status);
    console.log(response.statusText);
    console.log(response.headers);
    console.log(response.config);
  });
```
在项目中没有使用统一的错误处理机制，所以错误处理参考：[错误处理](https://www.axios-http.cn/docs/handling_errors)
