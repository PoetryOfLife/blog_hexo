---
title: JWT
date: 2022/11/31 09:46:30
tags: 
 - MySQL
categories: 
 - MySQL
---





## 1. 摘要

JSON Web Token（缩写 JWT）是目前最流行的跨域认证解决方案，本质就是一个字符串书写规范，作用是用来在用户和服务器之间传递安全可靠的信息。







## 2. JWT是什么

根据维基百科的定义，JSON WEB Token（JWT，读作 [/dʒɒt/]），是一种基于JSON的、用于在网络上声明某种主张的令牌（token）。JWT通常由三部分组成: 头信息（header）, 消息体（payload）和签名（signature）。

在目前前后端分离的开发过程中，使用token鉴权机制用于身份验证是最常见的方案，流程如下

![Image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bc95b7fab854121900bfa9db432d1c3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

1. 假设用户要登录一个APP，用户就需要输入用户名和密码，然后发送给APP的服务器
2. 服务器验证过用户发来的用户名和密码后，就会生成一个token，
3. 服务端返回JWT信息给用户，JWT中
4. 用户发送请求的时候一般会在HTTP头部加上这个Token
5. 服务器收到Token后，对Token进行核实，Token验证通过，且没有过期
6. 用户直接登录，不需要再次输入用户名密码，服务器只需识别头部的Token就可以确认用户身份

这里生成的Token就用的JWT这种数据结构

✒️Token的携带 token可以放在Cookie，Authorization，或者Body里面



## 3. JWT 的数据结构

Token，分成了三部分，头部（Header）、载荷（Payload）、签名（Signature），并以.进行拼接。其中头部和载荷都是以JSON格式存放数据，只是进行了base64 编码(secret 部分是否进行 base64 编码是可选的，header 和 payload 则是必须进行 base64 编码)，由于编码过程是可逆的，如果得知编码方式后，那么整个 jwt 串便是明文了，所以pyaload中一定不能放密码等重要信息。

![Image [2].png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a979f6cbd9e48eda11c79268ad975f4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### header

Token，分成了三部分，头部（Header）、载荷（Payload）、签名（Signature），并以.进行拼接。其中头部和载荷都是以JSON格式存放数据，只是进行了base64 编码(secret 部分是否进行 base64 编码是可选的，header 和 payload 则是必须进行 base64 编码)，由于编码过程是可逆的，如果得知编码方式后，那么整个 jwt 串便是明文了，所以pyaload中一定不能放密码等重要信息。

![Image [2].png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a979f6cbd9e48eda11c79268ad975f4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### header

头部主要是用来指明签名的算法，避免消息被篡改，JWT 中常用的签名算法是 HS256，常见的还有md5,sha 等，签名算法是不可逆的。声明算法的字段名为alg，同时还有一个typ的字段，默认JWT即可。以下示例中算法为HS256。

```javascript
{"alg": HS256, "typ": "JWT"}
```

#### payload

负载主要是用来存放数据，一般可以存放相应用户数据来生成不同的JWT

```javascript
"payload": {
        "data": [
          {
            "tooltt": "https://tooltt.com"
          }
        ],
        "iat": 1650451633,
        "exp": 1650556799
  }
```

JWT规定了7个官方字段，供选用

- iss (issuer)：签发人
- exp (expiration time)：过期时间
- sub (subject)：主题
- aud (audience)：受众
- nbf (Not Before)：生效时间
- iat (Issued At)：签发时间
- jti (JWT ID)：编号

除了官方字段，你还可以在这个部分定义私有字段，下面就是一个例子。

```javascript
{
      "sub": "1234567890",
      "name": "John Doe",
      "admin": true
}
```

由于编码的可逆性，不要把秘密信息放在这个部分。

#### Signature

签名是对头部和负载两个部分进行签名，防止数据篡改。 签名里面有个核心就是要定义一个密钥，这个密钥只有服务器能知道，然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。 公式如下：

```go
token, err = tokenClaims.SignedString([]byte(JwtKey))
```

一旦前面两部分数据被篡改，只要服务器加密用的密钥没有泄露，得到的签名肯定和之前的签名不一致

### 验证流程：

- 在头部信息中声明加密算法和常量，然后把header使用json转化为字符串
- 在载荷中声明用户信息，同时还有一些其他的内容，再次使用json把在和部分进行转化，转化为字符串
- 使用在header中声明的加密算法来进行加密，把第一部分字符串和第二部分的字符串结合和每个项目随机生成的secret字符串进行加密，生成新的字符串，此字符串是独一无二的
- 解密的时候，只要客户端带着jwt来发起请求，服务端就直接使用secret进行解密，解签证解出第一部分和第二部分，然后比对第二部分的信息和客户端穿过来的信息是否一致。如果一致验证成功，否则验证失败。



### 两种登录状态

#### 有状态登录

为了保证客户端cookie的安全性，服务端需要记录每次会话的客户端信息，从而识别客户端身份，根据用户身份进行请求的处理，典型的设计如tomcat中的session。

例如登录：用户登录后，我们把登录者的信息保存在服务端session中，并且给用户一个cookie值，记录对应的session。然后下次请求，用户携带cookie值来，我们就能识别到对应session，从而找到用户的信息。

缺点是什么？

- 服务端保存大量数据，增加服务端压力
- 服务端保存用户状态，无法进行水平扩展
- 客户端请求依赖服务端，多次请求必须访问同一台服务器
- 即使使用redis保存用户的信息，也会损耗服务器资源。

#### 无状态登录

微服务集群中的每个服务，对外提供的都是Rest风格的接口。而Rest风格的一个最重要的规范就是：服务的无状态性，即：

服务端不保存任何客户端请求者信息
客户端的每次请求必须具备自描述信息，通过这些信息识别客户端身份

带来的好处是什么呢？

- 客户端请求不依赖服务端的信息，任何多次请求不需要必须访问到同一台服务
- 服务端的集群和状态对客户端透明
- 服务端可以任意的迁移和伸缩
- 减小服务端存储压力

无状态登录的流程：

- 当客户端第一次请求服务时，服务端对用户进行信息认证（登录）
- 认证通过，将用户信息进行加密形成token，返回给客户端，作为登录凭证
- 以后每次请求，客户端都携带认证的token
- 服务的对token进行解密，判断是否有效。



### 单点登录

##### Jwt + 认证中心redis + 多系统redis

```
1.用户去认证中心登录，认证中心生成jwt，保存到redis并返回给客户端。
  	2.客户端携带jwt去多个系统认证  
    3.多系统(比如系统A)收到jwt，A解析并取出用户信息，先判断自己的A的redis中有没有jwt。
    	3.1 如果有，就合法，a系统可以继续执行业务逻辑。
        3.2 如果没有就拿着jwt去认证中心验证。
        	3.2.1 如果通过，a系统就把这个jwt保存到自己的redis，并设置对应的失效时间。
        	      下次这个jwt再来到a的时候，就不需要去认证中心校验了。
          	3.2.2 如果验证不通过此次请求就不合法，告诉客户端需要跳转登录页面，
           		  去认证中心登录，返回步骤1。
       

优点：安全性高，平均认证过程较快。

缺点：服务端流程复杂，需要考虑jwt的同步问题。比如注销或重新登录后，认证中心删除旧jwt需要同步给其他系统，其他系统删除自己保存的jwt。
```



### JWT的优缺点

#### 优点

- JWT具有通用性，所以可以跨语言组成简单，
- 字节占用小，便于传输服务端无需保存会话信息，
- 很容易进行水平扩展一处生成，多处使用，
- 可以在分布式系统中，
- 解决单点登录问题可防护CSRF攻击

#### 缺点

- payload部分仅仅是进行简单编码，所以只能用于存储逻辑必需的非敏感信息
- 需要保护好加密密钥，一旦泄露后果不堪设想
- 为避免token被劫持，最好使用https协议



### 解决单点登录问题可防护CSRF攻击

**CSRF攻击-其他页面窃取cookie**

用户访问A网站([http://www.aaa.com](https://link.zhihu.com/?target=http%3A//www.aaa.com))，输入用户名密码

服务器验证通过，生成sessionid并返回给客户端存入cookie

用户在没有退出或者没有关闭A网站，cookie还未过期的情况下访问恶意网站B

B网站返回含有如下代码的html：

//假设A网站注销用户的url为：[https://www.aaa.com/delete_user](https://link.zhihu.com/?target=https%3A//www.aaa.com/delete_user)



token验证过程

用户访问网站，输入账号密码登入

服务器校验通过，生成JWT，不保存JWT，直接返回给客户端

客户端将JWT存入cookie或者localStorage

此后用户发起的请求，都将使用js从cookie或者localStorage读取JWT放在http请求的header中，发给服务端

**服务端获取header中的JWT，用base64URL算法解码各部分内容，并在服务端用同样的秘钥和算法生成signature，与传过来的signature对比，验证JWT是否合法**

使用JWT验证，由于服务端不保存用户信息，不用做sessonid复制，这样集群水平扩展就变得容易了。同时用户发请求给服务端时，前端使用JS将JWT放在header中手动发送给服务端，服务端验证header中的JWT字段，而非cookie信息，这样就避免了CSRF漏洞攻击。

不过，无论是cookie-session还是JWT，都存在被XSS攻击盗取的风险：



### 白名单机制

在中间件中执行，借鉴gin的路由前缀树思想，将所有的API路由构建成前缀树，通过请求的URL进行匹配，如果在白名单数组中接口可以直接跳过登录鉴权。
