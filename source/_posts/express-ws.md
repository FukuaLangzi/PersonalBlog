---
title: express-generator模板与express-ws的不兼容问题
date: 2024-08-29 17:50:30
tags:
  - express
  - express-ws
  - bug
description: express-generator模板与express-ws的不兼容问题
---

## 由于最近在做一个 express 的项目，需要实时向前端传输 pm2 的 log 日志，于是想到了用 websocket，但是在使用时遇到了一些问题。

### 参考链接：https://github.com/HenningM/express-ws/issues/104

### 接下来从头开始，创建一个 express-generator 模板的项目并添加 websocket

1. 创建项目

```powershell
express express-websocket
cd express-websocket
npm i
code .
```

2. 引入 express-ws

```powershell
npm i express-ws
```

3. 先启动项目看能否正确运行

```powershell
nodemon npm run start
```

nodemon 没有的可以安一下

出现下面的就是启动成功了
{% asset_img 1.png %}
如果想加点提示可以在/bin/www 这个文件中更改

```javascript
function onListening() {
  var addr = server.address();
  var bind = typeof addr === "string" ? "pipe " + addr : "port " + addr.port;
  debug("Listening on " + bind);
  console.log(`${addr.port}端口监听中`); // 这里加一句话就行
}
```

加完之后要重新运行 nodemon npm run start,因为这个文件属于配置文件没有被监听

之后按照 express 文档的做法，添加 express-ws 的包

```javascript
var expressWs = require("express-ws");
var app = express();
expressWs(app); // 这句话一定要在use路由之前
```

然后在 express 的各个小 router 中也要做 ws 的处理

```javascript
var express = require("express");
var expressWs = require("express-ws");
var router = express.Router();
expressWs(router);
```

然后就可以正常书写 ws 代码了

```javascript
// wsmodule.js
router.ws("/test", async function (ws, req, next) {
  console.log("connect success");
  console.log(ws);

  // 使用 ws 的 send 方法向连接另一端的客户端发送数据
  ws.send("connect to express server with WebSocket success");

  // 使用 on 方法监听事件
  //   message 事件表示从另一段（服务端）传入的数据
  ws.on("message", function (msg) {
    console.log(`receive message ${msg}`);
    ws.send("default response");
  });

  // 设置定时发送消息
  let timer = setInterval(() => {
    ws.send(`interval message ${new Date()}`);
  }, 2000);

  // close 事件表示客户端断开连接时执行的回调函数
  ws.on("close", function (e) {
    console.log("close connection");
    clearInterval(timer);
    timer = undefined;
  });
});
```

这时启动代码就不会有报错，但是发送请求的时候会 404，我们还需要修改一个地方

```javascript
// bin/www
/**
 * Create HTTP server.
 */

var server = http.createServer(app);
// 把上面的话转换成下面的
var server = app;
```

这样就可以成功连接了,地址是：ws://localhost:3000/wsmodule/test

具体原因看 issue 就行
