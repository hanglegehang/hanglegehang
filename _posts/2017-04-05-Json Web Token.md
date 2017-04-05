---
layout: post
title:  "Json Web Token的定义和使用环境"
subtitle:   ""
author:     "李航"
date:   2017-04-05 12:20:00 +0800
catalog: true
tags: Jwt
---
### **什么是Json Web Token?**

 - Json Web Token （JWT）是一种开放标准（RFC7519），它定义了一种简单且独立的方式，用于将各方之间的信息安全地传输为JSON对象。该信息可以通过数字签名进行验证和信任。JWT可以使用一个secret（HMAC算法）或使用RSA的公钥/私钥对可以对JWT进行签名。
 - 进一步解释这个定义的一些概念。
  - **紧凑：**由于其较小的尺寸，JWT可通过URL，POST参数或HTTP标头内发送。另外，较小的尺寸意味着传输速度很快。
  - **独立的：**有效载荷包含有关用户的所有必需信息，避免了多次查询数据库的需要。

### **什么时候应该使用JSON Web令牌？**

 - **认证：**这是使用JWT最常见的情况。一旦用户登录，每个后续请求将包括JWT，允许用户访问该Token允许的路由，服务和资源。单点登录是一个广泛使用JWT的功能，因为它的开销很小，并且能够在不同的域中轻松使用。
 - **信息交换：** JSON Web令牌是在各方之间安全传输信息的好方法，因为它可以签名，例如使用公钥/私钥对，您可以确定发件人的真实性。另外，当使用header和payload计算签名时，您还可以验证内容是否未被篡改。

### **JSON Web Token的结构**

>  一个JWT实际上就是一个字符串，它由三部分组成，头部（Header）、载荷（Payload）与签名（Signature）。
> JWT通常如下所示：xxxxx.yyyyy.zzzz

#### 头部(Header)
Header通常由两部分组成：token的类型，即JWT，以及使用的哈希算法，如HMAC SHA256或RSA。
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```
进行Base64编码，之后的字符串就成了JWT的Header（头部）：
#### 载荷（Payload）
token的第二部分是payload，包含了我们声明的属性(claims)，claims是关于实体(通常是用户)和附加元数据的声明。claims有三种类型：reserved, public, 和private。我们可以先将用户认证的操作描述成一个JSON对象。其中添加了一些其他的信息，帮助今后收到这个JWT的服务器理解这个JWT。
**1、Reserved claims:** 这是一组预定义的不强制使用但是推荐使用的属性，
```
{
    "sub": "1",
    "iss": "http://localhost:8000/auth/login",
    "iat": 1451888119,
    "exp": 1454516119,
    "nbf": 1451888119,
    "jti": "37c107e4609ddbcc9c096ea5ee76c667"
}
```
这里面的前6个字段都是由JWT的标准所定义的。

 - sub: 该JWT所面向的用户
 - iss: 该JWT的签发者
 - iat(issued at): 在什么时候签发的token
 - exp(expires): token什么时候过期
 - nbf(not before)：token在此时间之前不能被接收处理
 - jti：JWT ID为web token提供唯一标识
 
**Public claims:** 这些可以由使用JWT的人随意定义
**Private claims:** 

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
将上面的JSON对象进行base64编码形成JSON Web Token的第二部分。Base64是一种编码，也就是说，它是可以被翻译回原来的样子来的。它并不是一种加密过程。

#### (签名)Signature
签名部分需要使用经过base64编码的Header和经过base64编码的载荷和一个`secret`，使用在Header中定义的算法加密

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```
加密后得到的字符串就是token的第三部分。签名(Signature)用于验证JWT的发件人的真实性，并确保该消息没有改变。

### **完整的JWT**
完整的tiken是三个Base64编码的字符串，以点分隔，可以轻松地在HTML和HTTP环境中传递，而与基于XML的标准（如SAML）相比，它们更便于传输。

关于用java如何实现JWT的，我会在今后的文章为大家说明

>参考文献：[http://www.tuicool.com/articles/IRJnaa](http://www.tuicool.com/articles/IRJnaa)
>
>[https://jwt.io/introduction/](https://jwt.io/introduction/)