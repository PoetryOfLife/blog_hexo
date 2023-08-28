jwt token 永不过期_ShawshanLin 的博客 - CSDN 博客







### 目录

- [JWT](https://blog.csdn.net/weixin_46120706/article/details/126945267#JWT_1)
- - [1. 什么是 JWT？](https://blog.csdn.net/weixin_46120706/article/details/126945267#1JWT_2)
  - - [签名的作用？](https://blog.csdn.net/weixin_46120706/article/details/126945267#_12)
  - [2.JWT 如何在分布式场景下进行用户认证的？](https://blog.csdn.net/weixin_46120706/article/details/126945267#2JWT_15)
  - - [方案一：网关统一校验](https://blog.csdn.net/weixin_46120706/article/details/126945267#_16)
    - [方案二：应用认证方案](https://blog.csdn.net/weixin_46120706/article/details/126945267#_18)
    - [两种方案对比](https://blog.csdn.net/weixin_46120706/article/details/126945267#_20)
  - [3. 无状态的 JWT 如何实现续签的功能？](https://blog.csdn.net/weixin_46120706/article/details/126945267#3JWT_28)
  - - [3.1 为什么 JWT 要续签？](https://blog.csdn.net/weixin_46120706/article/details/126945267#31JWT_29)
    - [3.2 不允许改变 Token 令牌的情况下实现续签](https://blog.csdn.net/weixin_46120706/article/details/126945267#32Token_35)
    - [3.3 允许改变 JWT 实现续签](https://blog.csdn.net/weixin_46120706/article/details/126945267#33JWT_38)
    - - [3.3.1 续签时重发 JWT 问题解决](https://blog.csdn.net/weixin_46120706/article/details/126945267#331JWT_42)





# JWT



## 1. 什么是 JWT？



Json Web Token（JWT）是一个经过加密的，包含用户信息的且具有时效性的固定格式字符串。

![img](https://img-blog.csdnimg.cn/2626955b9b5544209bbb9f8dd6268f7c.png#pic_center)


三部分字符串分别是标头、载荷和签名，即 **标头. 载荷. 签名**。





- 标头：原始数据是一个 json 字符串，字符串内容是 加密算法（alg）和类型（typ），用 base64 编码。

  ![img](https://img-blog.csdnimg.cn/026a60b294c44093bbf5431d84a56191.jpeg#pic_center)

- 载荷：也是一个 json 字符串，包含着所属用户的非敏感信息，也用 base64 来编码。

  ![img](https://img-blog.csdnimg.cn/24dea7ae85ff4100a0d6365819f0ce36.jpeg#pic_center)

- 签名：拿前面两部分的内容用服务器的密钥进行算法加密。

  ![img](https://img-blog.csdnimg.cn/cef46434363543a4be097b872c61a032.jpeg#pic_center)



### 签名的作用？



这个签名是用来做[数据校验](https://so.csdn.net/so/search?q=数据校验&spm=1001.2101.3001.7020)的，用户获取到 JWT 后，后面的每一次访问都会带上 JWT，服务器会对这个 JWT 进行校验，对 JWT 的前两个部分用服务器私钥做算法（HS256）的加密，得到加密后的结果后与用户 JWT 的第三部分相比较，只有两者完全相同的情况下，才会认为这个 JWT 是一个合法合规的，不是人为篡改或者造假的令牌。



## 2.JWT 如何在分布式场景下进行用户认证的？



### 方案一：网关统一校验





![img](https://img-blog.csdnimg.cn/4c0dbc45edd4416785f9283d0af03947.jpeg#pic_center)





### 方案二：应用认证方案





![img](https://img-blog.csdnimg.cn/5ed1e8a152da41608f108b868cb1e9a8.jpeg#pic_center)





### 两种方案对比



- 方案一：
  - 优点：JWT 校验无感知，校验与业务相分离（无代码侵入）。
  - 缺点：执行效率低，无论请求哪一个接口都需要先去校验取权限，适用于对效率要求不高的企业级应用。
- 方案二：
  - 优点：控制更加灵活，适用于追求性能的互联网应用。
  - 缺点：有代码侵入。



## 3. 无状态的 JWT 如何实现续签的功能？



### 3.1 为什么 JWT 要续签？



如果给 JWT 设定了一个有效时间的话，时间到 JWT 是会过期的。但假如这个时候用户正在投简历，填了一大堆信息，正准备提交呢，突然退回登录页，如果你是用户，你也会很不爽吧（狗头）。这个时候我们就需要对 JWT 进行续签。
但是，在原始 JWT 设计下，是没有考虑续签问题的。所以续签（即延长 JWT 的过期时间）工作需要我们自己来做。



不懂就问：



- **JWT 不设置过期时间行不行？** 答：不行，这会留下太空垃圾，后患无穷。



### 3.2 不允许改变 Token 令牌的情况下实现续签





![img](https://img-blog.csdnimg.cn/b7c1ed2da584425b9ddbc49353719fe9.jpeg#pic_center)





### 3.3 允许改变 JWT 实现续签





![img](https://img-blog.csdnimg.cn/c3b8801fae3643628e42c08a4a753fe0.jpeg#pic_center)



![img](https://img-blog.csdnimg.cn/fc2cf61588134dc2b05b7a978ccee4ec.jpeg#pic_center)





#### 3.3.1 续签时重发 JWT 问题解决



- 认证中心设计一个计时 Map 数据结构
- 只记录过去 n 秒内的原始 jwt 刷新所生成新 jwt 数据
- 几秒内如果发现同样的 jwt 在再次请求刷新，就返回相同的新 jwt 数据