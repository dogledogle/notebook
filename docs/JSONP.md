JSONP（**JSON with Padding**）是一个在 Web 开发史上非常有名的“奇招”。虽然现在它已经逐渐被更规范、更安全的 **CORS**（跨源资源共享）所取代，但理解它的原理对于掌握浏览器安全机制和 Web 演进过程非常有帮助。

---

## 1. 为什么需要 JSONP？（背景）

在浏览器中，出于安全考虑，存在**同源策略（Same-Origin Policy）**。简单来说，如果你的网站是 `a.com`，浏览器会限制你通过 AJAX（XMLHttpRequest 或 Fetch API）去请求 `b.com` 的数据。

但是，开发者发现 HTML 中有几个标签是不受同源策略限制的：
* `<script src="...">`
* `<img src="...">`
* `<link href="...">`

**JSONP 的核心思想：** 既然 `<script>` 标签可以跨域加载 JS 文件，那我们能不能让服务器动态生成一段 JS 代码，把数据“塞”进这段代码里送回来？

---

## 2. JSONP 的工作原理

JSONP 的实现分为三步：

### 第一步：客户端定义回调函数
在发起请求前，本地先声明一个处理数据的函数。
```javascript
function handleResponse(data) {
    console.log("收到数据啦！", data);
}
```

### 第二步：客户端动态插入 `<script>` 标签
请求的 URL 中通常会带上一个参数（约定俗称叫 `callback`），告诉服务端函数名。
```html
<script src="https://api.example.com/getUser?id=123&callback=handleResponse"></script>
```

### 第三步：服务端包装数据并返回
服务端接收到请求后，取到 `callback` 的值 `handleResponse`，然后将数据包装成一个**函数调用**的形式返回。

**服务端返回的内容：**
```javascript
handleResponse({ "name": "张三", "age": 25 });
```

当浏览器下载完这段 JS 后，会立即执行它。因为 `handleResponse` 已经在本地定义好了，所以数据就这样被成功传入了本地函数。

---

## 3. JSONP 与 AJAX 的区别

| 特性 | JSONP | AJAX (CORS) |
| :--- | :--- | :--- |
| **原理** | 动态插入 `<script>` 标签 | 使用 `XMLHttpRequest` 或 `Fetch` |
| **HTTP 方法** | **仅支持 GET** | 支持 GET, POST, PUT, DELETE 等 |
| **兼容性** | 极好（老旧浏览器全支持） | 现代浏览器支持（IE10+） |
| **安全性** | 较低（容易受 XSS 攻击） | 较高（有完善的权限控制） |
| **错误处理** | 很难捕获 404 或 500 错误 | 有完善的错误处理机制 |

---

## 4. 优缺点分析

### 优点
1.  **跨域无阻：** 在 CORS 普及之前，它是跨域的唯一解。
2.  **兼容性无敌：** 哪怕是 IE6 也能跑得飞起。

### 缺点
1.  **只能发 GET 请求：** 因为 `<script>` 标签的 `src` 只能发送 GET 请求。你没法通过 JSONP 提交一个复杂的表单或上传文件。
2.  **安全隐患（XSS）：** 如果服务端被攻击，返回了恶意脚本，客户端会直接执行。
3.  **缺乏状态码：** 如果服务器挂了返回 500，`<script>` 标签通常不会报错，很难定位问题。

---

## 5. 总结

JSONP 本质上是一个**利用标签漏洞的“黑客式”方案**。

* **什么时候用？** 除非你需要支持极其古老的浏览器（如 IE8 及以下），或者对接的极其老旧的第三方 API 只提供 JSONP 接口。
* **现代方案是什么？** 绝大多数情况下，请直接使用 **CORS**。只需在服务端设置一个响应头：
    `Access-Control-Allow-Origin: *`
    这比 JSONP 更安全、更强大、更优雅。
