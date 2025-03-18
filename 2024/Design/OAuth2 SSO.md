
2024-03-27 11:14

SSO（Single sign-on）：在多个相互信任的应用系统中，只需要登录一次，就可以访问其他的应用系统。



[[toc]]

---
---


---
https://www.jianshu.com/p/95598407fb90

# SSO & CAS

无论是SSO还是普通登录，它们的实现都是基于Cookie-Session模式的
基本流程：
- 第一次请求，服务器创建session返回sessionid
- 后续的请求 cookie中 带上 sessionid

cookie 是==和 域名关联==的。
有如下重要的属性
- Name，单条cookie信息的key值，通过这个key 设置和获取 存储在 cookie中的数据
- Value，我们所说的cookie的值
- Expires/Max-Age，cookie记录的到期时间
- Domain，cookie的==作用范围==

## 各子系统在相同域名下

一般情况下，企业只有一个 二级域名，然后通过三级域名区分不同的系统。
比如:
- 公司的二级域名是 .bl.com，
- 2个业务系统是: .app1.bl.com, .app2.bl.com，
- 登录系统: .sso.bl.com

我们要实现 在 .sso.bl.com 登录， .app1.bl.com, .app2.bl.com 也登录

用户通过 浏览器登录 .sso.bl.com，根据 cookie-session模式可知，.sso.bl.com 的服务端的 session 中 会记录登录状态，同时 浏览器会 保存 cookie信息，其中 cookie的 Domain属性 是 .sso.bl.com

要让其他2个系统 登录，需要解决2个问题
1. 浏览器的cookie的 Domain 是 .sso.bl.com，在发给 app1.bl.com, app2.bl.com 的请求中无法携带
2. sso,app1,app2 是不同的应用，session在它们各自的服务器中，不共享

针对第一个问题，sso登录后，可以将 cookie 的 Domain 设置为 二级域名 .bl.com，这样 所有子域的系统都可以访问到 二级域的 cookie。
第二个问题，解决方法就是 session 共享

。。修改cookie的Domain通常是指在==服务器端修改==发送到客户端的==Set-Cookie头==中的domain属性
。。session共享：数据库，redis，spring session，反向代理

## 各子系统属于不同的域

不同域，无法共享cookie

新的解决方案就是 
### 单点登录

SSO的标准流程就是 CAS

CAS， Central Authentication Service ——— 中央认证服务，是 Yale 大学发起的一个企业级的、开源的项目

![eedf280e3fe4bdc4c0e7c9c153fe4e57.png](../_resources/eedf280e3fe4bdc4c0e7c9c153fe4e57.png)


第一次访问APP
- 用户访问APP，app需要登录，app验证cookie 和session ，发现用户没有登录，要求 用户 302 临时重定向到 CAS server，同时要修改 service 参数
- CAS server 发现 用户没有登录，弹出用户登录页
- 用户填写 用户，密码， CAS server 确认后，CAS server 生成一个 该用户的 SSO session 存放在服务器本地session中，并生成一个 SSO session id，即 TGT。随后重定向返回一个 CASTGC (放在cookie中) 给用户浏览器，这个 TGC中包含了TGT的值
- CAS server 登录完成后，会生成一个 ST (service ticket) 返回给浏览器，然后 浏览器 被 302 跳转到 app系统，同时将 ST 作为参数传递给 app 系统。
- app系统拿到 ST后，从后台 向 CAS 发送请求，验证 ST是否有效
- 验证通过后，app 将登录状态写入 session 并设置 app域下的cookie，再返回给浏览器。
- 用户发起请求，带上cookie，app系统校验 cookie和session。

第二次访问app
- 用户发起请求，带上cookie中的 jsessionid 给app服务器
- app服务器 使用 jsessionid 和 session 中存储的数据 进行校验


第一次访问app2
- 访问 app2 服务，用户未登录，重定向到 CAS server
- 浏览器 访问 CAS server，发送认证请求，带上 TGC信息
- CAS server 通过 TGC中的 TGT 查找 SSO session 的 信息进行认证
- 认证通过，生成 ST， 重定向 浏览器 到 app2 的服务地址
- app2服务 发送ST给 CAS server 进行认证
- CAS server 认证成功
- app2服务 设置session，并重定向到 app2 的地址，并设置 cookie MOD_AUTH_CAS_S
- 浏览器发起请求，带上cookie的 MOD_AUTH_CAS_S，发给app2
- app2使用 MOD_AUTH_CAS_S 和 session中的信息进行校验

## CAS重要字段

TGT， ticket granting ticket
CAS server 创建 TGT，存放在 CAS server的 session中，根据用户信息签发。
简单来说，TGT就是 SSO session 的ID

TGC, ticket granting cookie
创建TGT的时候，生成TGC，通过 CAS server的 response header 的 set-cookie 字段设置 TGC，其中包含了 TGT的信息

ST, service ticket
根据TGT 签发的 ST，是 CAS 为用户签发的 访问 某一 service 的票据。在CAS server 创建 TGT 后，生成一个 ST 将其添加到 重定向地址后。

ST是随机生成的。 ST只能存活一定的时间，CAS server 会令其失效。
ST只能用一次。




---

# OAuth 2.0

四种授权方式

## 授权码
最常用，第三方应用先申请一个授权码，然后再用该码获取令牌。

1. 网站A 提供链接，用户点击后 跳转到 B网站，希望 授权用户数据 给 A网站使用
```uri
https://b.com/oauth/authorize?
  response_type=code&		// code代表授权码
  client_id=CLIENT_ID&		// 让B知道是谁在请求
  redirect_uri=CALLBACK_URL&	// B接收或拒绝后 跳转的网址
  scope=read	// 授权范围
```

2. 跳转到 B网站后，用户登录，B询问用户是否 同意授权给 A。同意，则跳转到 `redirect_uri` 中的 A网站，并带上 授权码
```uri
https://a.com/callback?code=AUTHORIZATION_CODE
```

3. A 拿到授权码后，就可以在后端，向B 请求令牌
```uri
https://b.com/oauth/token?
 client_id=CLIENT_ID&		// 让B 确认A的身份
 client_secret=CLIENT_SECRET&	// 同上。
 grant_type=authorization_code&	// 授权码方式
 code=AUTHORIZATION_CODE&	// 第二步获得的 授权码
 redirect_uri=CALLBACK_URL	// 令牌颁发后的回调网址
```

4. B 收到请求后，就颁发令牌，向 `redirect_uri` 的网址，发送一段JSON数据，JSON中的 access_token 就是 令牌。A就拿到了
```JSON
{
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```


## 隐藏式
允许直接向前端颁发令牌。没有授权码这个中间步骤。一般用于 纯前端项目

纯前端，就必须把令牌存储在前端。

1. A提供一个链接，点击后，跳转到B，希望 授权用户数据给A使用
```uri
https://b.com/oauth/authorize?
  response_type=token&		// 要求直接返回 令牌
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

2. 用户跳转到B后，同意授权。B跳会 `redirect_uri`，并把令牌 作为 url参数
```uri
https://a.com/callback#token=ACCESS_TOKEN
```

3. A 从url参数中拿到 令牌。上面url中 是 锚点(#)，不是查询字符串

浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险

很不安全。

所以有效期短。


## 密码式
直接通过 用户名 密码 申请令牌，最不安全

如果你高度信任A，那么可以告诉 网站A，你的网站B的账号密码，网站A就使用你的账号密码 去 网站B 申请令牌。

1. A要求用户提供 B的账号密码，拿到后，A直接向B请求令牌
```uri
https://oauth.b.com/token?
  grant_type=password&	// 密码式
  username=USERNAME&	// B的用户名 和 密码
  password=PASSWORD&
  client_id=CLIENT_ID
```

2. B身份验证通过后，直接给出令牌。 这里不需要跳转，令牌直接放在 ==JSON数据==中，作为==HTTP 响应==，A拿到令牌


## 凭证式
针对第三方app，而不是针对用户，即某个第三方app的所有用户共用一个令牌，一般用户 后台api 服务消费者设计。

适用于没有前端的 命令行应用

1. A在命令行向B发出请求
```url
https://oauth.b.com/token?
  grant_type=client_credentials&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```

3. B验证通过后，直接返回令牌。


## 令牌的使用

A拿到令牌后，就可以向 B 的API 发送请求了。
每个请求都要在http header带上令牌
```shell
curl -H "Authorization: Bearer ACCESS_TOKEN" \
"https://api.b.com"
```

## 更新令牌

OAuth 2.0 允许用户自动更新令牌

B颁发令牌的时候，一次性颁发2个，一个用于获取数据，另一个用于获取新的令牌 (refresh token字段)。令牌到期前，用户使用 refresh token 发送一个请求，去更新令牌。
```url
https://b.com/oauth/token?
  grant_type=refresh_token&		// 更新令牌
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET&	// 确认身份
  refresh_token=REFRESH_TOKEN
```





