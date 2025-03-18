Restful-api
2021年8月24日
13:39
https://github.com/bolasblack/http-api-guide

如果请求头中存在 X-HTTP-Method-Override 或参数中存在 _method（拥有更高权重），且值为 GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD 之一，则视作相应的请求方式进行处理

GET, DELETE, HEAD 方法，参数风格为标准的 GET 风格的参数，如 url?a=1&b=2

POST, PUT, PATCH, OPTIONS 方法

    - 默认情况下请求实体会被视作标准 json 字符串进行处理，当然，依旧推荐设置头信息的 Content-Type 为 application/json
    - 在一些特殊接口中（会在文档中说明），可能允许 Content-Type 为 application/x-www-form-urlencoded 或者 multipart/form-data ，此时请求实体会被视作标准 POST 风格的参数进行处理

    - OPTIONS  用于获取资源支持的所有 HTTP 方法
    - HEAD  用于只获取请求某个资源返回的头信息
    - GET 用于从服务器获取某个资源的信息
        - 完成请求后返回状态码 200 OK
        - 完成请求后需要返回被请求的资源详细信息
    - POST 用于创建新资源
        - 创建完成后返回状态码  201 Created
        - 完成请求后需要返回被创建的资源详细信息
    - PUT 用于完整的替换资源或者创建指定身份的资源，比如创建 id 为 123 的某个资源
        - 如果是创建了资源，则返回 201 Created
        - 如果是替换了资源，则返回 200 OK
        - 完成请求后需要返回被修改的资源详细信息
    - PATCH 用于局部更新资源
        - 完成请求后返回状态码 200 OK
        - 完成请求后需要返回被修改的资源详细信息
    - DELETE 用于删除某个资源
        - 完成请求后返回状态码  204 No Content

输出的数据结构中空字段的值一律为 null
国际化-语言标签

[RFC 5646](http://tools.ietf.org/html/rfc5646) ([BCP 47](http://tools.ietf.org/html/bcp47)) 规定的语言标签的格式如下：

language-script-region-variant-extension-privateuse
状态码  -  请求成功

- 200  **OK**  : 请求执行成功并返回相应数据，如 GET 成功
- 201  **Created**  : 对象创建成功并返回相应资源数据，如 POST 成功；创建完成后响应头中应该携带头标 Location ，指向新建资源的地址
- 202  **Accepted**  : 接受请求，但无法立即完成创建行为，比如其中涉及到一个需要花费若干小时才能完成的任务。返回的实体中应该包含当前状态的信息，以及指向处理状态监视器或状态预测的指针，以便客户端能够获取最新状态。
- 204  **No Content**  : 请求执行成功，不返回相应资源数据，如 PATCH ， DELETE 成功

**重定向的新地址都需要在响应头 Location 中返回**

- 301  **Moved Permanently**  : 被请求的资源已永久移动到新位置
- 302  **Found**  : 请求的资源现在临时从不同的 URI 响应请求
- 303  **See Other**  : 对应当前请求的响应可以在另一个 URI 上被找到，客户端应该使用 GET 方法进行请求。比如在创建已经被创建的资源时，可以返回 303
- 307  **Temporary Redirect**  : 对应当前请求的响应可以在另一个 URI 上被找到，客户端应该保持原有的请求方法进行请求

**条件请求**

- 304  **Not Modified**  : 资源自从上次请求后没有再次发生变化，主要使用场景在于实现[数据缓存](https://github.com/bolasblack/http-api-guide#user-content-%E6%95%B0%E6%8D%AE%E7%BC%93%E5%AD%98)
- 409  **Conflict**  : 请求操作和资源的当前状态存在冲突。主要使用场景在于实现[并发控制](https://github.com/bolasblack/http-api-guide#user-content-%E5%B9%B6%E5%8F%91%E6%8E%A7%E5%88%B6)
- 412  **Precondition Failed**  : 服务器在验证在请求的头字段中给出先决条件时，没能满足其中的一个或多个。主要使用场景在于实现[并发控制](https://github.com/bolasblack/http-api-guide#user-content-%E5%B9%B6%E5%8F%91%E6%8E%A7%E5%88%B6)

**客户端错误**

- 400  **Bad Request**  : 请求体包含语法错误
- 401  **Unauthorized**  : 需要验证用户身份，如果服务器就算是身份验证后也不允许客户访问资源，应该响应 403 Forbidden 。如果请求里有 Authorization 头，那么必须返回一个  [WWW-Authenticate](https://tools.ietf.org/html/rfc7235#section-4.1)  头
- 403  **Forbidden**  : 服务器拒绝执行
- 404  **Not Found**  : 找不到目标资源
- 405  **Method Not Allowed**  : 不允许执行目标方法，响应中应该带有 Allow 头，内容为对该资源有效的 HTTP 方法
- 406  **Not Acceptable**  : 服务器不支持客户端请求的内容格式，但响应里会包含服务端能够给出的格式的数据，并在 Content-Type 中声明格式名称
- 410  **Gone**  : 被请求的资源已被删除，只有在确定了这种情况是永久性的时候才可以使用，否则建议使用 404 Not Found
- 413  **Payload Too Large**  : POST 或者 PUT 请求的消息实体过大
- 415  **Unsupported Media Type**  : 服务器不支持请求中提交的数据的格式
- 422  **Unprocessable Entity**  : 请求格式正确，但是由于含有语义错误，无法响应
- 428  **Precondition Required**  : 要求先决条件，如果想要请求能成功必须满足一些预设的条件

405。。。这个是指GET，POST这种方法不允许。
406，客户端的http头中有一个accept字段，表示客户端  需要的返回内容的格式，  如果服务器没有办法生成这种格式，就406.
415，客户端的Content-Type头，表示了  请求体的  格式，如果服务器不能处理这种格式，就415
422，一切没有问题，但是服务器端  无法理解  客户端的请求体（语义错误）
**服务端错误**

- 500  **Internal Server Error**  : 服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。
- 501  **Not Implemented**  : 服务器不支持当前请求所需要的某个功能。
- 502  **Bad Gateway**  : 作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。
- 503  **Service Unavailable**  : 由于临时的服务器维护或者过载，服务器当前无法处理请求。这个状况是临时的，并且将在一段时间以后恢复。如果能够预计延迟时间，那么响应中可以包含一个 Retry-After 头用以标明这个延迟时间（内容可以为数字，单位为秒；或者是一个  [HTTP 协议指定的时间格式](http://tools.ietf.org/html/rfc2616#section-3.3)）。如果没有给出这个 Retry-After 信息，那么客户端应当以处理 500 响应的方式处理它。

501 与 405 的区别是：405 是表示服务端不允许客户端这么做，501 是表示客户端或许可以这么做，但服务端还没有实现这个功能

REST 服务的要求之一就是[超文本驱动](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)，客户端不再需要将某些接口的 URI 硬编码在代码中，唯一需要存储的只是 API 的 HOST 地址，能够非常有效的降低客户端与服务端之间的耦合，服务端对 URI 的任何改动都不会影响到客户端的稳定。

目前有几种方案试图实现这个效果：

- [JSON HAL](http://tools.ietf.org/html/draft-kelly-json-hal-07)  ，示例可以参考  [JSON HAL 作者自己的介绍](http://stateless.co/hal_specification.html)
- [GitHub API 使用的方案](https://developer.github.com/v3/#hypermedia)  ，应该是一种 JSON HAL 的变体
- [JSON API](http://jsonapi.org/)  ，（这里有  [@迷渡](https://github.com/justjavac)  发起的  [中文版](http://jsonapi.org.cn/)  ），另外一种类似 JSON HAL 的方案
- [Micro API](http://micro-api.org/)  ，一种试图与  [JSON-LD](http://json-ld.org/)  兼容的方案

目前所知的方案都实现了发现资源的功能，服务端同时需要实现 OPTIONS 方法，并在响应中携带 Allow 头来告知客户端当前拥有的操作权限。

。。。就是只需要知道host+端口，剩下的url，能实时计算出来。
**数据缓存**

大部分接口应该在响应头中携带 Last-Modified, ETag, Vary, Date 信息，客户端可以在随后请求这些资源的时候，在请求头中使用 If-Modified-Since, If-None-Match 等请求头来确认资源是否经过修改。

如果资源没有进行过修改，那么就可以响应 304 Not Modified 并且不在响应实体中返回任何内容。
**并发控制**

不严谨的实现，或者缺少并发控制的 PUT 和 PATCH 请求可能导致 “更新丢失”。这个时候可以使用 Last-Modified 和/或 ETag 头来实现条件请求，支持乐观并发控制。

下文只考虑使用 PUT 和 PATCH 方法更新资源的情况。

- 客户端发起的请求如果没有包含  If-Unmodified-Since 或者 If-Match 头，那就返回状态码  403 Forbidden  ，在响应正文中解释为何返回该状态码
- 客户端发起的请求提供的 If-Unmodified-Since 或者 If-Match 头与服务器记录的实际修改时间或 ETag 值不匹配的时候，返回状态码 412 Precondition Failed

。。。上面的不匹配  似乎是指  值不是历史值也不是现在的值。还得保存历史值？  还是说  412是指  无法解析值。感觉是  无法解析  值。但是无法解析，应该是  400？请求体无法解析是400，但是  请求头中的字段  无法解析  是什么。。也是400？

- 客户端发起的请求提供的 If-Unmodified-Since 或者 If-Match 头与服务器记录的实际修改时间或 ETag 的历史值匹配，但资源已经被修改过的时候，返回状态码 409 Conflict
- 客户端发起的请求提供的条件符合实际值，那就更新资源，响应 200 OK 或者 204 No Content ，并且包含更新过的 Last-Modified 和/或 ETag 头，同时包含 Content-Location 头，其值为更新后的资源 URI

**CORS**
接口支持[“跨域资源共享”（Cross Origin Resource Sharing, CORS）](http://www.w3.org/TR/cors)
**JSON-P**
如果在任何 GET 请求中带有参数 callback ，且值为非空字符串，那么接口将返回如下格式的数据
。。。似乎是把  首页，前一页，后一页，末页，的uri  返回给了  客户端。

已使用 Microsoft OneNote 2016 创建。