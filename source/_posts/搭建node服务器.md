---
title: 搭建node服务器
date: 2021/09/25
tags: ["nodejs", "服务器"]
---

{% blockquote %}
{% endblockquote %}

## 准备工作

- 安装 node，打开 cmd，**node -v**查看 node 版本
- node 安装成功，即可通过 node index.js 运行 js 文件，但是改动 js 文件，需要重新启动，服务才能生效
- 安装 nodexmon **npm install -g nodemon**，可实现自动更新文件

## 搭建 webServer

```js
// index.js
const http = require("http");
// 创建server对象
// const server = new http.Server()
const server = http.createServer();
// 注册request事件回调函数，当有请求时触发
server.on("request", () => {
  console.log("有客户请求");
});
server.listen("3000", "0.0.0.0", (err) => {
  if (err) console.log("服务启动失败");
  console.log("服务启动成功");
});
```

启动命令**nodemon index.js**,在浏览器输入 localhost:3000,request 事件回调将能监听到请求

## 返回响应数据

在每一次的 request 事件中回调函数中会传递两个参数：IncomingMessage，ServerResponse
通过 ServerResponse 向客户端返回数据

```js
const server = http.createServer();
server.on("request", (req, res) => {
  console.log("有客户请求");
  // res.end('hello world')
  res.write("hello world");
  res.end();
});
server.listen("3000", "0.0.0.0", (err) => {
  if (err) console.log("服务启动失败");
  console.log("服务启动成功");
});
```

参考：https://nodejs.org/dist/latest-v15.x/docs/api/http.html#http_class_http_serverresponse

## 获取请求 URL、读取静态资源

使用 IncomingMessage 获取与客户端相关的信息,通过 fs 读取文件

- 在 index 同级目录创建 resource 文件夹

```text
  ├─ index.js
  └─ resource
    ├─ a.html
    └─ b.html
```

```js
// index.js
const http = require("http");
const fs = require("fs");
const server = http.createServer();
server.on("request", (req, res) => {
  // req.url ：端口号以后的路径
  try {
    const content = fs.readFileSync(`./resource${req.url}`);
    res.end(content);
  } catch (err) {
    res.end(JSON.stringify("Not found"));
  }
});
server.listen("3000", "0.0.0.0", (err) => {
  if (err) console.log("服务启动失败");
  console.log("服务启动成功");
});
```

即可在浏览器地址栏中输入 localhost:3000/a.html 获取到服务端的 a.html 文件。
通过 req.url 需要自己解析路径参数，可以通过 url 模块，将 req.url 解析为对象

```js
// index.js
const http = require("http");
const fs = require("fs");
const url = require("url")
const server = http.createServer();
server.on("request", (req, res) => {
  // req.url ：端口号以后的路径 /a.html?name=hah
  const urlObj = url.parse(req.url)
  try{
    const content = fs.readFileSync(`./resource${urlObj.pathname}`)
    res.end(content)
  }catch(err){
    res.end(JSON.stringify('Not found'))
  }
});
...
/*
// urlObj
{
  protocol: null,
  slashes: null,
  auth: null,
  host: null,
  port: null,
  hostname: null,
  hash: null,
  search: '?name=hah',
  query: 'name=hah',
  pathname: '/a.html',
  path: '/a.html?name=hah',
  href: '/a.html?name=hah'
}
*/
```

```js
// 通过路径，匹配不同资源
if (urlObj.pathname.startsWith("/resource")) {
  try {
    const content = fs.readFileSync(`.${urlObj.pathname}`);
    res.end(content);
  } catch (err) {
    res.end(JSON.stringify("Not found"));
  }
} else {
  res.end(Math.random());
}
```

url 模块参考：https://nodejs.org/api/url.html

## 设置响应头

### Content-Type

针对不同类型的资源，为了接收方知道当前接收的内容类型，以便针对该类型进行对应处理，所以需要在发送响应前，应告知正文内容类型格式.

MIME:是用来表示文档、文字的性质和格式的标准，通用结构：type/subtype.

```js
// index.js同级目录创建mimeType.js 处理文件类型

const mimeType = {
  css: "text/css",
  js: "application/javascript",
  json: "application/json",
  html: "text/html",
  txt: "text/plain",
  jpeg: "image/jpeg",
  jpg: "image/jpg",
  gif: "image/gif",
  png: "image/png",
  webp: "image/webp",
  mp4: "video/mp4",
  mp3: "audio/mp3",
};
//找到地址对应的文件类型
function matchMimeTypes(url) {
  //获取元素地址的后缀名
  const arr = url.split(".");
  const extName = arr[arr.length - 1].toLowerCase();
  return mimeType[extName] || "text/plain";
}
module.exports = matchMimeTypes;
```

```js
const http = require("http");
const fs = require("fs");
const url = require("url");
// 引入
const matchMimeTypes = require("./mimeType");
const server = http.createServer();
server.on("request", (req, res) => {
  const urlObj = url.parse(req.url);
  // 设置Content-Type
  const mimeType = matchMimeTypes(urlObj.pathname);
  res.setHeader("Content-Type", `${mimeType};charset=utf-8`);
  ...
});
```

### 状态码 statusCode

- 信息响应(100–199)
- 成功响应(200–299)
- 重定向(300–399)
- 客户端错误(400–499)
- 服务器错误 (500–599)

```js
res.statusCode = 200;
```

## 动态资源 动态路由表

```js
// index.js同级目录 routesMap.js路由表
const routesMap = new Map();
routesMap.set("/", async (req, res) => {
  res.setHeader("Content-Type", 'text/html;charset="utf-8"');
  res.end("首页...");
});
routesMap.set("/list", async (req, res) => {
  res.setHeader("Content-Type", 'text/html;charset="utf-8"');
  res.end("列表...");
});
module.exports = routesMap;
```

```js
const http = require("http");
const fs = require("fs");
const url = require("url");
const matchMimeTypes = require("./mimeType");
const routesMap = require("./routesMap");
// 创建server对象
const server = http.createServer();
// 注册request事件回调函数，当有请求时触发
server.on("request", (req, res) => {
  const urlObj = url.parse(req.url);
  console.log(urlObj, "urlObj1");
  // 静态资源
  if (urlObj.pathname.startsWith("/resource")) {
    const mimeType = matchMimeTypes(urlObj.pathname);
    res.setHeader("Content-Type", `${mimeType};charset=utf-8`);
    try {
      const content = fs.readFileSync(`.${urlObj.pathname}`);
      res.end(content);
    } catch (err) {
      res.end(JSON.stringify("Not found"));
    }
  } else {
    try {
      const routeHandle = routesMap.get(urlObj.pathname);
      routeHandle(req, res);
    } catch (err) {}
  }
});
server.listen("3000", "0.0.0.0", (err) => {
  if (err) console.log("服务启动失败");
  console.log("服务启动成功");
});
```

## 请求重定向

通过设置状态码 302，和 location

```js
const http = require("http");
const fs = require("fs");
const url = require("url");
const matchMimeTypes = require("./mimeType");
const routesMap = require("./routesMap");
const server = http.createServer();
server.on("request", (req, res) => {
  const urlObj = url.parse(req.url);
  console.log(urlObj, "urlObj1");
  // 静态资源
  if (urlObj.pathname.startsWith("/resource")) {
    const mimeType = matchMimeTypes(urlObj.pathname);
    res.setHeader("Content-Type", `${mimeType};charset=utf-8`);
    try {
      const content = fs.readFileSync(`.${urlObj.pathname}`);
      res.end(content);
    } catch (err) {
      res.statusCode = 404;
      res.end("资源不存在");
    }
  } else {
    try {
      const routeHandle = routesMap.get(urlObj.pathname);
      routeHandle(req, res);
    } catch (err) {
      // 当静态资源读取失败，返回首页
      res.statusCode = 302;
      res.setHeader("Location", "/");
      res.end();
    }
  }
});
server.listen("3000", "0.0.0.0", (err) => {
  if (err) console.log("服务启动失败");
  console.log("服务启动成功");
});
```

## KOA

基于 NodeJS 的 web 框架，致力于 web 应用和 API 开发。
