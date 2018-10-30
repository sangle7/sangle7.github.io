---
title: 理解 JSON Web Tokens (JWT) 的 5 个简单步骤
date: 2018-09-28 10:14:44
tags: 
  - 网络
  - 翻译
  - JWT
---

> 原文链接：https://medium.com/vandium-software/5-easy-steps-to-understanding-json-web-tokens-jwt-1164c0adfcec

在本文中，将解释JSON Web Tokens（JWT）的基本原理以及使用原因。 JWT 是确保应用程序信任和安全的重要部分。 JWT 允许以安全的方式表示诸如用户数据之类的声明。

为了解释 JWT 如何工作，让我们从一个抽象的定义开始。

> JSON Web令牌（JWT）是一种 [JSON 对象](http://www.w3schools.com/json/)，在 [RFC 7519](https://tools.ietf.org/html/rfc7519) 中定义为在两方之间表示一组信息的安全方式。 JWT 由头部（header），负载（payload）和签名（signature）组成。


简单地说，JWT只是一个具有以下格式的字符串：

```
header.payload.signature
```

*应该注意，双引号字符串被认为是有效的 JSON 对象。*

<!-- more -->

为了说明实际使用 JWT 的方式和原因，我们将使用一个简单的示例（参见下图）。 此示例中的三个不同的实体是用户，应用程序服务器和身份验证服务器。 验证服务器将向用户提供 JWT。 使用 JWT，用户可以安全地与应用程序通信。

![img](https://cdn-images-1.medium.com/max/1600/1*SSXUQJ1dWjiUrDoKaaiGLA.png)

应用程序如何使用JWT验证用户的真实性



在该示例中，用户首先使用认证服务器的登录系统登录认证服务器（例如，用户名和密码，Facebook登录，Google登录等）。 然后，身份验证服务器创建 JWT并将其发送给用户。 当用户对应用程序进行 API 调用时，用户将传递 JWT 以及 API 调用。 在这个实例中，应用程序服务器将可以验证传入的 JWT 是否是由身份验证服务器创建的（验证过程将在稍后更详细地说明）。当用户使用附加的 JWT 进行API 调用时，应用程序可以使用 JWT 来验证该 API 调用是否来自经过身份验证的用户。

现在，将更深入地研究 JWT 本身及其构建和验证的方式。



## Step 1. 创建 Header

JWT 的 Header 部分包含有关如何计算 JWT 签名的信息，是一个以下形式的 JSON 对象：

```JSON
{
    "typ": "JWT",
    "alg": "HS256"
}
```

在上面的 JSON 中，“typ”键的值指定对象是JWT，“alg”键的值指定用于创建 JWT 签名的算法。 在示例中，我们使用 HMAC-SHA256算法（一种使用密钥的散列算法）来计算签名（在步骤3中会更详细的介绍）。

## Step 2. 创建 PAYLOAD

JWT 的 payload 部分时是存储在 JWT 内的数据。在我们的示例中，身份验证服务器创建一个JWT，其中存储有用户信息，特别是用户ID。

```json
{
    "userId": "b08f86af-35da-48f2-8fab-cef3904660bd"
}
```

The data inside the payload is referred to as the “claims” of the token.

在我们的示例中，我们只将一个声明放入 payload 中。 你可以根据需要添加任意数量的声明。JWT 规定了7个官方字段，供选用。

```
iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号
```

除了官方字段，你还可以在这个部分定义私有字段。请记住，数据的大小将影响JWT的总体大小，这通常不是问题，但过大的 JWT 可能会对性能产生负面影响并导致延迟。

## Step 3. 创建 SIGNATURE

签名使用以下伪代码计算：

```
// signature algorithm
data = base64urlEncode( header ) + “.” + base64urlEncode( payload )
hashedData = hash( data, secret )
signature = base64urlEncode( hashedData )
```

该算法所做的是 base64url 对在步骤1和2中创建的header和payload进行编码。然后，算法将得到的编码字符串用“点”（.）连在一起。

在示例中，header 和 payload 被 base64url 编码为：

```
// header
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
// payload
eyJ1c2VySWQiOiJiMDhmODZhZi0zNWRhLTQ4ZjItOGZhYi1jZWYzOTA0NjYwYmQifQ
```

然后，在加入周期的编码头和编码有效载荷上应用带有密钥的指定签名算法，我们得到签名所需的散列数据。 在我们的例子中，这意味着在数据字符串上应用 HS256 算法，并将密钥设置为字符串“secret”，以获取 hashedData字符串。 之后，通过base64url 编码 hashedData 字符串，我们得到以下JWT签名：

```
// signature
-xN_h82PHVTCMA9vdoHrcZxH-x5mb11y1537t3rGzcM
```

## Step 4. 把 JWT 的三个部分组合在一起

现在我们已经创建了所有三个组件，我们可以创建JWT。 记住JWT的header.payload.signature结构，我们只需要组合以上的三个部分，用点（.）分隔它们。

```
// JWT Token
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOiJiMDhmODZhZi0zNWRhLTQ4ZjItOGZhYi1jZWYzOTA0NjYwYmQifQ.-xN_h82PHVTCMA9vdoHrcZxH-x5mb11y1537t3rGzcM
```

你可以尝试通过 [jwt.io](http://jwt.io/) 创建自己的JWT。

回到我们的示例，身份验证服务器现在可以将此 JWT 发送给用户。

### JWT 如何保护我们的数据？

要理解使用 JWT 的目的不是以任何方式隐藏或模糊数据，而是为了证明发送的数据是由真实的来源创建的。

如前面的步骤所示，JWT 内的数据是经过编码和签名的，而不是加密的。 编码数据的目的是转换数据的结构。 签名数据允许数据接收器验证数据源的真实性。 因此，编码和签名数据不会保护数据。 另一方面，加密的主要目的是保护数据并防止未经授权的访问。 有关编码和加密之间差异的更详细说明，请参阅[此文章](https://danielmiessler.com/study/encoding-encryption-hashing-obfuscation/#encoding)。

> 由于 JWT 仅被签名和编码，并且由于 JWT 未加密，因此 JWT 不能保证敏感数据的安全性。

## Step 5. 校验 JWT

在我们的示例中，我们使用的是由 HS256 算法签名的JWT，其中只有身份验证服务器和应用服务器知道密钥。当应用程序设置其身份验证过程时，应用程序服务器从身份验证服务器接收密钥。由于应用程序知道密钥，当用户对应用程序进行带有 JWT 附加的 API 调用时，应用程序可以执行与 JWT 上的步骤3相同的签名算法。然后，应用程序可以验证从其自己的哈希操作获得的签名是否与 JWT 本身上的签名匹配（即，它与由认证服务器创建的 JWT 签名匹配）。如果签名匹配，则表示 JWT 有效，表示 API 调用来自可信源。否则，如果签名不匹配，则表示收到的 JWT 无效，这可能是对应用程序的潜在攻击的指示。因此，通过验证 JWT，应用程序在其自身和用户之间添加了一层信任。

## 结论

我们了解了 JWT 是什么，如何创建和验证它们，以及如何使用它们来确保应用程序与其用户之间的信任。这是了解 JWT 基础知识及其有用之处的起点。 JWT 只是确保应用程序中的信任和安全性的难题之一。
应该注意，本文中描述的 JWT 身份验证设置使用对称密钥算法（HS256）。你也可以以类似的方式设置 JWT 身份验证，除非使用非对称算法（例如RS256），其中身份验证服务器具有密钥，并且应用程序服务器具有公钥。查看此 [Stack Overflow问题](https://stackoverflow.com/questions/39239051/rs256-vs-hs256-whats-the-difference)，了解使用对称和非对称算法之间差异的详细分类。
还应该注意，JWT 应该通过 HTTPS 连接发送。拥有 HTTPS 有助于防止未经授权的用户通过使用它来窃取所发送的 JWT，从而无法拦截服务器和用户之间的通信。
此外，在 JWT 中设置较短的过期时间十分重要，这样如果旧的 JWT 受到盗用，它们将被视为无效并且不能再使用。