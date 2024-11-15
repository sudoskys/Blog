---
title: Web 开发中的 OAUTH 实现
date: 2024-11-16 12:00:00
tags:
  - Web
---

建议先读完 JWT 的文章，再来看这篇文章。

## 解决跨应用认证问题

针对跨应用的认证问题，SAML 协议是一个解决方案，但是 SAML 太复杂了，而且 XML 不利于传输和解析。

所以 OAuth 2.0 和 OIDC(OpenID Connect) 出现了。

OIDC 是基于 OAuth 2.0 的身份认证协议，针对移动互联网进行了优化，它是一个简单的身份层。

OIDC在令牌上，不仅支持 SAML Assertion，更是支持具有JSON格式的JWT。有了JWT加持，OIDC协议比SAML更易于传输和使用。

### 关于 OAuth 2.0

认证除了在一个应用系统内进行，还有一个问题是在不同的应用系统之间进行认证。

比如用户在 A 系统登录后，想要访问 B 系统的资源，这时候就涉及到如何在 A 系统和 B 系统之间传递用户的登录凭证。

但是如果直接传递用户名和密码，就会有很大的安全风险。

所以人们想出了一种授权机制，叫做 OAuth（Open Authorization）。

OAuth 2.0 是一个开放标准，它允许用户授权第三方应用访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方应用。

OAuth 2.0 的核心思想是授权，而不是认证。最常见的 OAuth 2.0 授权方式是授权码模式（Authorization Code Grant）和密码模式（Resource Owner Password Credentials Grant）。

### 授权码模式

授权码模式是 OAuth 2.0 的标准授权方式，它是一个三方授权的过程：
