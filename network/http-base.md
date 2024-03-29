# HTTP 协议

## HTTP 各个版本之间的简要区别

HTTP（Hypertext Transfer Protocol）是一种用于在客户端和服务器之间传输超文本的协议。以下是 HTTP 各个版本之间的主要区别：

1. HTTP/0.9：这是最早的 HTTP 版本，于 1991 年发布，仅支持 GET 方法，并且没有任何 HTTP 头部信息。它主要用于传输简单的 HTML 文档。

2. HTTP/1.0：于 1996 年发布，引入了多种请求方法（GET、POST、HEAD），支持响应状态码、HTTP 头部字段和 MIME 类型。但每个请求/响应周期都需要单独建立 TCP 连接，性能较差。

3. HTTP/1.1：于 1999 年发布，是当前广泛使用的 HTTP 版本。相比 HTTP/1.0，它引入了持久连接(Connection: keep-alive)、管线化、分块传输编码和缓存控制等特性，提高了性能和效率。

   - host 头部字段：HTTP/1.1 引入了 host 头部字段，使得一台服务器可以托管多个域名。例如，可以在同一台服务器上托管www.example.com和www.example.net两个域名，而不需要为每个域名分配一个IP地址。

4. HTTP/2：于 2015 年发布，是对 HTTP/1.1 的重大改进。它采用了二进制协议，实现了多路复用，即在同一个连接上可以同时进行多个请求/响应交互。此外，还引入了头部压缩、服务器推送和流优先级等功能，进一步提升了性能和效率。

5. HTTP/3：正在进行标准化，基于 QUIC（Quick UDP Internet Connections）协议。HTTP/3 通过在 UDP 上运行，实现了更低的延迟和更好的性能，尤其在高丢包率的网络环境中表现出色。

这些 HTTP 版本之间的改进主要关注于性能、效率和安全性。从 HTTP/1.0 到 HTTP/1.1 的改进使得连接复用成为可能，减少了建立连接的开销；HTTP/2 引入了多路复用，提高了并发性能；HTTP/3 通过 QUIC 协议进一步减少了延迟。同时，各个版本也对安全性进行了不断的增强，如引入 HTTPS 的普及和加密算法的改进。

选择使用哪个 HTTP 版本取决于应用程序的需求和兼容性要求。大多数现代浏览器和服务器都支持 HTTP/1.1 和 HTTP/2，并逐渐开始支持 HTTP/3，以提供更好的性能和安全性。

## 缓存

### 强制缓存

设置强缓存可以通过在服务器端的响应头中添加特定的缓存控制字段来实现。以下是两个常用的设置强缓存的方法：

1. 使用"Expires"头部字段（已过时）：

   - 在服务器端的响应头中添加"Expires"字段，该字段指定了资源的过期时间。例如，设置为一个未来的日期，如："Expires: Wed, 02 Jun 2023 12:00:00 GMT"。
   - 客户端浏览器在接收到该响应后，会将该资源缓存，并在过期时间之前直接从缓存中获取，而不向服务器发送请求。

2. 使用"Cache-Control"头部字段：
   - 在服务器端的响应头中添加"Cache-Control"字段，该字段提供了更灵活的缓存控制选项。
   - 设置"Cache-Control"字段的值为"max-age"，后跟表示缓存有效期的秒数。例如，设置为一天的秒数： "Cache-Control: max-age=86400"。
   - 客户端浏览器在接收到该响应后，会将该资源缓存，并在指定的时间内直接从缓存中获取。

请注意，"Cache-Control"头部字段优先于"Expires"头部字段，因为它提供了更精确的控制。使用"Cache-Control"字段更为常见和推荐。

例如，以下是使用"Cache-Control"头部字段设置强缓存的示例代码（使用 Node.js 和 Express 框架）：

```javascript
app.get("/example", function (req, res) {
  // 设置缓存有效期为一天
  res.setHeader("Cache-Control", "max-age=86400");

  // 其他响应设置...

  // 返回响应
  res.send("Hello, World!");
});
```

通过设置适当的缓存控制头部字段，可以让浏览器在有效期内直接从缓存中获取资源，从而提高页面加载性能和用户体验。

"Cache-Control"头部字段用于控制缓存的行为，它可以接受多个指令，并以逗号分隔。下面是一些常见的"Cache-Control"指令及其含义：

1. public：表示响应可以被任何缓存（包括客户端浏览器和共享代理服务器）缓存。

2. private：表示响应只能被客户端浏览器缓存，不允许共享代理服务器进行缓存。适用于包含个人用户数据的响应，如用户登录后的页面。

3. no-cache：表示客户端缓存和中间代理服务器在使用缓存之前必须先将请求提交给原始服务器进行验证。原始服务器可以通过返回特定的验证响应来决定是否要返回实际的资源内容。

4. no-store：表示不允许缓存对该响应进行存储，即每次请求都需要重新获取资源。

5. max-age=\<seconds\>：指定资源在缓存中的最大有效期限，以秒为单位。例如，"max-age=3600"表示资源可以在缓存中保存 1 个小时。

6. s-maxage=<seconds\>：类似于 max-age，但仅适用于共享代理服务器缓存。当存在 s-maxage 时，它会覆盖 max-age 指令。

7. must-revalidate：表示在资源过期后，缓存必须重新验证该资源的有效性。如果验证失败，则需要从原始服务器重新获取资源。

8. proxy-revalidate：类似于 must-revalidate，但仅适用于共享代理服务器缓存。

9. no-transform：表示中间代理服务器不得修改响应的媒体类型或内容。

这些指令可以单独使用，也可以组合使用，以满足特定的缓存策略需求。例如，"Cache-Control: public, max-age=3600"表示响应可以被任何缓存进行存储，并在缓存中保留 1 小时。

需要注意的是，不同的指令可以具有不同的优先级和互斥关系。因此，在设置"Cache-Control"头部字段时，需要仔细考虑每个指令的含义和适用场景，并根据具体需求进行选择和组合。

[MDN Cache-Control](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)

### 协商缓存

协商缓存是一种缓存机制，用于在客户端和服务器之间进行通信，以确定资源是否需要重新下载。相比于强缓存直接从缓存中获取资源，协商缓存需要与服务器进行一次请求和响应的交互。

协商缓存的实现基于两种机制：**实体标签（Entity Tag，ETag）**和**最后修改时间（Last-Modified）**。

1. 实体标签（ETag）：服务器可以为每个资源分配一个唯一的实体标签，该标签表示资源的特定版本。当客户端再次请求该资源时，客户端会将上一次获取的实体标签通过"If-None-Match"头部字段发送给服务器。服务器会比较客户端提供的实体标签与当前资源的实体标签是否匹配，如果匹配，服务器返回一个 304 Not Modified 的响应，指示客户端可以直接从缓存中获取资源，否则返回新的资源。

2. 最后修改时间（Last-Modified）：服务器可以在响应头部添加一个"Last-Modified"字段，该字段表示资源的最后修改时间。当客户端再次请求该资源时，客户端会将上一次获取的最后修改时间通过"If-Modified-Since"头部字段发送给服务器。服务器会比较客户端提供的最后修改时间与当前资源的最后修改时间是否匹配，如果匹配，服务器返回一个 304 Not Modified 的响应，指示客户端可以直接从缓存中获取资源，否则返回新的资源。

协商缓存的工作流程如下：

1. 客户端首次请求资源时，服务器会返回资源的完整内容，并在响应头部添加 **ETag** 和 **Last-Modified** 字段。

2. 客户端再次请求资源时，会在请求头部中携带上次获取的 ETag 和 Last-Modified 值。

3. 服务器通过比较请求头部中的 ETag 和 Last-Modified 值与当前资源的实体标签和最后修改时间来判断资源是否有更新。

4. 如果匹配，服务器返回 **304 Not Modified** 的响应，客户端可以直接从缓存中获取资源。

5. 如果不匹配，服务器返回新的资源，并在响应头部更新 ETag 和 Last-Modified 值。

需要注意的是，ETag 和 Last-Modified 可以单独或同时使用。一些高级的缓存策略可能使用更复杂的验证算法，如组合 ETag 和 Last-Modified，或使用其他自定义的验证机制。

协商缓存提供了更灵活的缓存控制方式，可以在资源未过期的情况下避免不必要的数据传输，减轻服务器负载，提高网络性能和用户体验。
