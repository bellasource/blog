---
title: KOA框架
date: 2021/09/26
tags: ["nodejs", "服务器", "KOA"]
---

## 准备工作

- npm init -y 初始化 npm 包
- npm i koa 安装 koa 包

参考文档：https://koa.bootcss.com/#context

## 搭建服务

```js
const Koa = require("koa");
// 创建Application对象
const app = new Koa();

// 通过use注册中间件
app.use((ctx) => {
  console.log("中间件", "1");
  ctx.body = "haha";
});
/* 
内部也是通过createServer创建服务，通过listen监听
http.createServer((req, res) => {})
*/
app.listen(8080);
```

- 中间件
  本质上就是用来完成各种具体业务逻辑的函数。Koa 的 Application 对象使用一个 middleware 来存储各种中间件函数

## 中间件

### 注册多个中间件

- context:
  - 通过 ctx 可以获取、设置 request，response，cookie 等
- next：
  - 包装了下一个中间件函数，只有调用 next，才能执行下一个中间件
  - next 方法总是返回一个 Promise 对象，可以处理异步操作

```js
const app = new Koa();
app.use((ctx, next) => {
  // 模拟用户登录数据
  const randomData = Math.random() * 10;
  if (randomData > 50) {
    ctx.body = "没有权限";
  } else {
    // next为下一个中间件注册的函数
    next();
    // 由于调用next，此时ctx.body 为"大海和小蕊的照片"
    ctx.body = `<h1>${ctx.body}</h1>`;
  }
});
app.use((ctx) => {
  ctx.body = "大海和小蕊的照片";
});
app.listen(8080);
```

### 常用中间件

- koa-static-cache：用来处理静态资源
- @koa/router：用来匹配路由
- koa-body:用于解析正文数据，并将数据转为对象存储在 ctx.request.body 中
- jsonwebtoken:跨域认证解决方案，用于生成、解析 token
- koa-server-http-proxy：用于转发请求
  安装：npm i koa-static-cache @koa/router koa-body

```text
├─ app.js
├─ package-lock.json
├─ package.json
├─ static
  └─ 1.html
  └─ 2.html
  └─ add.html
```

```js
const Koa = require("koa");
const koaStaticCache = require("koa-static-cache");
const KoaRouter = require("@koa/router");
const koabody = require("koa-body");

const app = new Koa();
// 模拟用户数据
let users = [
  { id: 1, username: "haizi" },
  { id: 2, username: "zMouse" },
];
// 处理静态资源中间件
app.use(koaStaticCache({ prefix: "/static", dir: "./static", gzip: true }));

// 处理路由中间件
const router = new KoaRouter();
router.get("/getusers", async (ctx, next) => {
  ctx.body = users;
});
router.post("/add", koabody(), async (ctx, next) => {
  users.push({ id: Date.now(), username: ctx.request.body.username });
  ctx.body = "添加成功";
  console.log(users);
});
// 使用路由中间件
app.use(router.routes());

app.listen(8080);
```

```html
<!-- add.html -->
<form action="/add" method="post">
  <p>用户名：<input type="text" name="username" /></p>
  <p><button>提交</button></p>
</form>
```
