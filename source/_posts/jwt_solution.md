---
title: Web 开发中的 JWT 实现
date: 2024-11-14 12:00:00
tags:
  - Web
cover: /img/cover/json_web_token.png
---

JWT（JSON Web Token）是一种用于在网络应用之间传递信息的简洁、自包含的方法。JWT 本质上是一个字符串，它由三部分组成，分别是 Header、Payload 和 Signature。在 Web 开发中，JWT 通常用于用户认证和授权。

先阅读一些 [基本概念](https://docs.authing.cn/v2/concepts/cryptography.html)

## 关于网络认证

> JSON Web Tokens are an open, industry standard RFC 7519 method for representing claims securely between two parties. --https://jwt.io/

在生活中，很多地方会用到身份验证和授权，比如进入公司大楼需要刷卡，进入银行需要提供身份证，进入酒吧需要出示年龄证明。

不过在网络世界中，HTTP 是无状态的，每次请求都是独立的，某段时间的两个请求，服务器不能知道两次请求是否来自同一个用户，所以我们需要一些信息确定这个请求是哪个用户发出的。

比如小明想访问邮箱，邮箱服务器需要知道是返回小明的邮件还是小红的邮件，所以需要知道小明的身份。

于是人们想出了两种思路来确定网络用户的身份：

1. 基于用户的属性，比如用户名和密码。
2. 给用户签发证件，用户登记自己的信息，服务器下发一个令牌，用户在每次请求时带上这个令牌。

于是大家就发明了很多种身份验证的方法。

### HTTP Basic Authentication

所以为了解决身份验证的问题，最原始的办法是在每次请求中都带上用户的登录凭证，比如用户名和密码。我们叫它 [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication)。

它是在 HTTP 1.0 中提出的，整个过程定义在 [RFC 2617](http://tools.ietf.org/html/rfc2617) 中，它的原理是在请求头中加入 `Authorization` 字段，值为 `Basic base64(username:password)`。服务器收到请求后，解码 `Authorization` 字段，就可以得到用户名和密码。参考[MDN HTTP Authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)

虽然基本认证非常容易实现，但是由于如果被**中间人截获**，就可以轻松解码出用户名和密码，所以不安全。

### Digest access Authentication

所以后来又出现了 [Digest access authentication](https://en.wikipedia.org/wiki/Digest_access_authentication)，详细规范在 [RFC 2069](http://tools.ietf.org/html/rfc2069)，它的原理是在请求头中加入 `Authorization` 字段，值为 `Digest username="username", realm="realm", nonce="nonce", uri="uri", response="response"`。服务器收到请求后，根据 `Authorization` 字段中的信息，生成一个随机数 `nonce`，然后将 `nonce` 发送给客户端，客户端将 `nonce` 和密码进行加密，然后再发送给服务器。服务器收到请求后，根据 `nonce` 和密码进行加密，然后比较两者是否相等。

```markdown
HTTP/1.0 401 Unauthorized Server: HTTPd/0.9 Date: Sun, 10 Apr 2014 20:26:47 GMT
WWW-Authenticate: Digest realm="testrealm@host.com", qop="auth,auth-int",
nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
opaque="5ccc069c403ebaf9f0171e9517f40e41" Content-Type: text/html
Content-Length: 153
```

不过我们很快发现一个问题：虽然流程很复杂，但是一点也不安全，因为**密码仍然可以被中间人截获或暴力破解**。因为它用的是 MD5，还很麻烦。

### App Secret Key + HMAC

[HMAC](https://en.wikipedia.org/wiki/HMAC) 定义于 [RFC 2104](http://tools.ietf.org/html/rfc2104)，是一种用于消息认证的算法，它结合了哈希算法和密钥，用来验证数据的完整性和真实性。

![wiki](https://upload.wikimedia.org/wikipedia/commons/thumb/0/08/MAC.svg/1322px-MAC.svg.png)

这个方法是在客户端和服务器端之间约定一个密钥，然后使用 HMAC 算法对数据进行签名。这样服务器端就可以验证数据是否被篡改。

比较好的例子是 AWS 的签名认证：

![hmac](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/images/sigV4-using-auth-header.png)

```markdown
Authorization: AWS4-HMAC-SHA256
Credential=AKIAIOSFODNN7EXAMPLE/20130524/us-east-1/s3/aws4_request, 
SignedHeaders=host;range;x-amz-date, 
Signature=fe5f80f77d5fa3beca038a248ff027d0445342fe2855ddc963176630326f1024
```

`Credential` 的形式是 `<your-access-key-id>/<date>/<aws-region>/<aws-service>/aws4_request`

收到经过身份验证的请求后，Amazon S3 服务器使用请求中包含的身份验证信息重新创建签名。如果签名匹配，Amazon S3 才会处理请求。

HMAC 的好处就是可以方便管理密钥，方便授权。而且可以通过加入时间戳或随机数来解决无法防止重放攻击问题。

不过问题又一次出现：HMAC 并没有统一的**实现标准**，而且 HMAC 似乎并不方便用于用户认证，因为 HMAC 本质上更适合授权。

## 浏览器的进步

浏览器也是一种客户端，它运行网页需要一种通用的和服务器交换数据的方式，**由于早期浏览器并没有现代存储 API**，所以 Cookie 就被发明出来存储数据，给每个网站一个交换数据的通道。

浏览器会存储 Cookie 并在下次向同一服务器再发起请求时携带并发送到服务器上。比如记录用户的登录状态。

Cookie 曾一度用于客户端数据的存储，当时并没有其他合适的存储办法而作为唯一的存储手段，但由于服务器指定 Cookie 后，浏览器的每次请求都会携带 Cookie 数据，会带来额外的性能开销，现在推荐使用现代存储 API。

**所以 Cookie 其实和认证没有直接关系，它只是一种存储机制。**

### 基于 Cookie 发明了 Session

Cookie 只是一个存储机制，浏览器在每次请求时都会带上这个文本，而且服务器可以通过设置 Cookie。Cookie 有很多属性，比如 `Domain`、`Path`、`Expires`、`HttpOnly`、`Secure` 等。

为了避免客户端每次都发送用户名和密码，人们结合 Cookie 发明了 Session。

Session 是*服务器端*的存储机制，服务器会生成一个唯一的 Session ID，然后将 Session ID 保存在 Cookie 中，每次用户请求时，服务器通过 Cookie 中的 Session ID 查找 Session 数据，从而判断用户是否登录。

只传递一个代表性的 ID，这样 Session 机制就能避免了用户密码泄露的问题。

### Session 的问题

单机情况下，Session 也有一些问题，它保存在数据库或内存里，如果我们有很多服务器，Session 的管理就会变得很复杂，如果遇到高并发，读写数据库的性能也会成为瓶颈。

当用户量增多，我们的服务器越来越多，经过水平扩展（把请求分流的不同的服务器处理），Session 的问题就凸显出来了。

Session 需要服务器保存供比对的信息，但是如果请求的服务器不是之前的服务器，那么 Session 就会失效。

不过也有解决办法：

- 粘性会话（Sticky Session）：在负载均衡器上设置，将同一个用户（一般根据IP）的请求都转发到同一个服务器上。但是负载均衡本身就是用于平衡负载的，如果一直转发到同一个服务器上，就失去了负载均衡的意义。

- Session 复制：将 Session 数据复制到所有服务器上，这样每个服务器都可以验证用户的 Session。不过跨服务器数据同步会引进大量问题。

- Session 集中存储：将 Session 数据存储在一个集中的地方，比如 Redis，这样每个服务器都可以访问 Session 数据。但是内存服务器下线整个系统就崩溃了（单点问题）。

### SSO 单点登录的出现解决了两个问题

[SSO](https://en.wikipedia.org/wiki/Single_sign-on)（Single Sign-On）是一种身份验证机制，允许用户使用一个身份验证凭证（比如用户名和密码）登录多个应用。

为了解决网页浏览器单点登录的需求，SAML（Security Assertion Markup Language）协议被提出，它是一种基于 XML 的标准，用于在不同的安全域之间交换身份和授权信息。

SAML 的核心思想是：用户登录后，身份提供者(IdP)给用户签发一个令牌，应用只需要验证令牌是否有效即可。不需要再查询数据库验证用户的用户名和密码。

验证令牌涉及到对称以及非对称加密算法，因此无关服务器。

之所以不需要查表验证用户密码，正是因为多台服务器持有一个密钥，请求内有原始数据和原始数据生成的签名，服务器通过密钥和原始数据生成签名，比对签名是否一致。如果一致就是自己人发出的请求。

不过 SAML 也有一些问题，比如 SAML 是基于 XML 的，XML 太复杂，而且 SAML 实现太复杂了。

不用 XML 肯定是要用 JSON，所以 Json Web Token 就出现了。用 Json 来做令牌，对移动互联网比较友好，传输也方便。

## JWT(JSON Web Token) 的出现

它可能是目前最流行的身份验证和授权解决方案。

JWT 是一种开放标准（[RFC 7519](https://tools.ietf.org/html/rfc7519)），它定义了一种简洁的、自包含的方法，用于在各方之间传递信息。

JWT 本质上是一个字符串，它由三部分组成，分别是 Header、Payload 和 Signature。

```json5
// eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
// Header
{
  "alg": "HS256",  // 指定签名算法
  "typ": "JWT"   // 固定值
}
// Payload
{
  "sub": "dianas.cyou",
  "name": "sudoskys",
  "iat": 1516239022
}
// Signature
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  your-256-bit-secret
)
```

### Payload 的结构

Payload 是 JWT 的第二部分，它包含了一些声明（Claim），Claim 是关于实体（通常是用户）和其他数据的声明。Claim 有三种类型：

1. Registered Claim：这些是一组预定义的声明，包括 iss（签发者）、sub（主题）、aud（受众）、exp（过期时间）、nbf（生效时间）、iat（签发时间）和 jti（JWT ID，用于防止重放攻击）。
2. Public Claim：这些是公共声明是预留用于行业共识的标准声明。
3. Private Claim：这些是自定义的声明，比如用户的 ID、姓名、角色等。

```json5
{
  "sub": "dianas.cyou", // Subject
  "name": "github@sudoskys",
  "admin": true,
  "iat": 1516239022 // Issued At
}
```

**由于 payload 是 base64 编码的，所以不要在 payload 中存储敏感信息。**

### Signature 的生成

为了确保信息不被篡改，JWT 还需要一个签名。签名是由 Header 和 Payload 以及一个密钥生成的。

```markdown
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  your-256-bit-secret
)
```

简单来说，就是将 Header 和 Payload 用 `.` 连接起来，然后用 HMACSHA256 算法进行加密，最后用 Base64 编码。

不过有的小朋友可能就要问了，JWT 和 普通的下发 Token 有什么区别呢？

### JWT 的特殊之处

首先我们要明确一个概念：Token 是一种通用的概念，它可以是任何形式的凭证，比如 JWT、OAuth 的 Access Token、Session ID 等，JWT 就是一种特定的 Token，它是一种开放标准，定义了一种简洁的、自包含的方法，用于在各方之间传递信息。所以 JWT 是 Token 的一种，而 Token 不一定是 JWT。

JWT 的优势在于它是**自包含**的，它包含了用户的信息，所以服务器不需要保存用户的登录状态，这样就可以实现无状态的认证。

这个设计非常聪明，什么是无状态的认证呢？

无状态的认证是指服务器不保存用户的登录状态，每次请求都是独立的，服务器不需要保存用户的登录状态，只需要验证 JWT 的有效性即可。

我们的 Header 和 Payload（Payload 不能放敏感信息哦） 都是 **BASE64编码** 的，Signature 是用 HMACSHA256 算法不可逆加密的。

所以如果服务器知道 JWT 签名加密的密钥，就可以验证 JWT 是否是自己签发的，如果签名匹配，那就证明这个请求是拥有密钥的服务器签发的。

而服务器不会签发非法的 JWT，所以 JWT 内的用户信息就是这个请求的用户信息。

>只要有了 jwt 相当于拿到账户权限，虽然编码是明文的，但是也是用户凭据，不可以公开到网上，JWT 本身就是用户的凭证。
> JWT 签发出来不可以被销毁，只能等待过期，因为 JWT 是无状态的，服务器不能控制用户的登录状态。不过可以把 JWT 加入黑名单，这样就可以实现过期功能。

而普通 Token 在服务器每次处理请求时都需要查询数据库，判断 Token 是否有效，而 JWT 只需要解码 JWT 即可。

如果我们有很多服务器，那么验证普通 Token 需要在每台服务器上查询数据库，而 JWT 只需要知道签发时的密钥进行比对即可。

而且 JWT 自带了过期时间，进一步增加了安全性。

### JWT 和 Cookie 的关系

JWT 是服务器签发给客户端的字符串，客户端在每次请求时都在请求头带上 JWT。但是客户端存储在哪里其实是没有限制的，JWT 可以存储在 Cookie、LocalStorage、SessionStorage 等地方。

JWT 可以存储在 Cookie 中（一般不这样做），也可以存储在 LocalStorage 或 [SessionStorage](https://zh.javascript.info/localstorage) 中。

不过因为 Cookie 有长度限制和安全问题（浏览器自动在普通请求中带上 Cookie，如果网站有漏洞，凭证就会被窃取），所以我们一般不把 JWT 存储在 Cookie 中，而是存储在 LocalStorage 或 SessionStorage 中。

### JWT 使用

客户端请求时在 `Authorization` 头中加入 `Bearer {jwt}` 字段，值为 JWT。

服务器根据 JWT 的过期时间和签名和密钥验证请求。

```typescript
// author:github@sudoskys
const jwt = require('jsonwebtoken');

// 验证 JWT
const token = req.headers.authorization.split(' ')[1];
jwt.verify(token, 'secret', (err, decoded) => {
  if (err) {
    res.status(401).send('Unauthorized');
  } else {
    res.send('Hello ' + decoded.name);
  }
});
```

一般用 [npm jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) 库来实现 JWT 的签发和验证。


JWT 无法销毁，只能等待过期，但是可以加入黑名单，这样就可以实现过期功能。

### 延续登录状态

你可以在每次请求时，请求接口检查JWT是不是快过期了，如果快过期了，就重新签发一个JWT，然后返回给客户端。

**不要签发一个永不过期的JWT。**

如果你的系统需要考虑安全性，我建议你添加 `Refresh Token` 机制，这样可以减少 JWT 被盗用的风险。

比如当你不小心把 JWT 泄漏到网上，但是你启用了 `Refresh Token` 机制，那么 JWT 很快就会过期，保证了安全性。

刷新令牌可能看起来是不必要的机制，因为当被中间人截获时，刷新令牌也会被截获。

主要就是防止某些意外情况下（比如使用了不规范的库，把你的 JWT 乱扔），你的 JWT 通过 Cookie 泄漏到网上造成大量损失。

