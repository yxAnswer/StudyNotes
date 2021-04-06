# 聊一聊用户权限校验那点事

[TOC]

## 1、单机和分布式应用的登录检验介绍

常见的登录验证和用户状态识别，发展有大概有以下几个。 

### 1.1  单机tomcat应用登录检验

在传统的单节点应用中，使用  sesssion 和cookie 的方式来进行校验。 sesssion保存在浏览器和应用服务器会话之间。用户登录成功，服务端会保证一个session，当然会给客户端一个sessionId，客户端会把sessionId保存在cookie中，每次请求都会携带这个sessionId

简单来说 cookie +session 方式

### 1.2 分布式应用中session共享和token出现

但是到了多个节点的分布式应用中，有很少节点，并且并发量不多的情况，就出现了session共享的方式。当然Tomcat可以开启session共享，但是不太好。因为

- **tomcat支持session共享**，但是有广播风暴；用户量大的时候，占用资源就严重，不推荐

所以并发量高的情况就出现了token验证，可以使用redis来存储token。

- **使用redis存储token:**
  服务端使用UUID生成随机64位或者128位token，放入redis中，然后返回给客户端并存储在cookie中
  用户每次访问都携带此token，服务端去redis中校验是否有此用户即可

虽然使用redis存储了token，解决了问题，但是每个节点每次校验都需要去和redis 做交互，会多花一点开销，在高并发的时候，网络io 的开销会浪费我们的带宽。所以还有一种主流的方式**jwt**。

## 2、分布式中的JWT 校验

### 2.1 JWT 介绍

什么是jwt，英文名(JSON Web Token )。官网地址：[https://jwt.io/](https://jwt.io/)

官方解释是这样的：

JSON Web Token (JWT)是一个开放标准(RFC 7519)，它定义了一种紧凑且自包含的方式，用于作为JSON对象在各方之间安全地传输信息。可以验证和信任此信息，因为它是数字签名的。JWTs可以使用秘密(使用HMAC算法)或使用RSA或ECDSA的公钥/私钥对进行签名。

简单来说，jwt就是通过加解密算法来进行身份状态校验的。通过一定规范来生成token，然后可以通过解密算法逆向解密token，这样就可以获取用户信息。

注意jwt是用来验证身份的，并不一定非要用于单点登录。

### 2.2 JWT工作流程：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190415081338157.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

1、客户端发出登录请求到服务端

2、服务端通过私钥、签名、加密算法生成一个token，

3、将token返回给客户端

4、客户端每次请求都要带着这个token

5、服务端每次接收请求都用解密算法来验证这个token

6、验证通过，将资源相应给客户端

### 2.3 JWT的优缺点

优点：

- 契合无状态api,并且体积小，传输快，支持跨域操作
- 存储在客户端，不占用服务端资源，有利于微服务
- 因为存储在客户端，所以分布式系统每个节点都能相应，都能验证状态，易实现CDN资源分布式管理
- 生成的token可以包含基本信息，比如id、用户昵称、头像等信息，避免再次查库

缺点：

 token是经过base64编码，所以可以解码，因此token加密前的对象不应该包含敏感信息，如用户权限，密码等

jwt主要应用于 分布式的无状态的api校验，所以对于有状态的个人觉得还是用redis+token吧。



### 2.4 JWT的组成

JWT 有三部分组成分别是，

- `Header`： 头部，主要是描述签名算法
-  `Payload`：负载，主要是描述加密对象信息，如用户id等
- `Signature`：签名,主要是把前面两部分进行加密，防止别人拿到token进行base解密后篡改token

最后形成类似这样

`xxxxxxxxx.yyyyyyyyyyy.zzzzzzz` 三部分

**(1) Header:**

Header是一段json对象，通常由两部分组成:

token的类型(JWT)和正在使用的签名算法(如HMAC SHA256或RSA)。例如：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

如上表明token 类型是JWT的，使用签名算法是HS256 。最终这段json会变为base64的token的第一部分

**(2) Payload:负载**

payload部分就是我们要传输的信息了。我们在这声明要传输的信息，一般分为三种。 There are three types of claims: *registered*, *public*, and *private* claims.  也就是

- 已注册的声明
- 公共的声明
- 私有的声明

**已注册的声明**： 是官方建议，但是不强制使用的声明，比如: **iss** (issuer), **exp** (expiration time), **sub** (subject), **aud**(audience), 其他。
`iss`: jwt签发者
`sub`: 面向的用户(jwt所面向的用户)
`aud`: 接收jwt的一方
`exp`: 过期时间戳(jwt的过期时间，这个过期时间必须要大于签发时间)
`nbf`: 定义在什么时间之前，该jwt都是不可用的.
`iat`: jwt的签发时间
`jti`: jwt的唯一身份标识，主要用来作为一次性`token`,从而回避重放攻击。

**公共部分的声明**：

这部分可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息。由于该部分在客户端可解密，所以不建议添加敏感信息。

**私有声明**：

这部分是为双方共享信息创建的，base64可对称解密，所以也不建议放敏感信息。

总之，这负载信息最终会变为base64生成token的中间部分。

**(3) Signature签名**

这部分是把前面两部分内容 进行加密。所以它有两个参数

第一个参数：经过 Base64 分别编码的 Header 及 Payload 通过 . 连接组成的字符串

第二个参数：生成的密钥，由服务器保存。

它是将前两部分通过base64后生成的字符串用`.` 连接起来，然后再通过header中声明的加密算法和秘钥进行加密生成。

最终，它变为jwt的第三部分。

jwt是存储在客户端的，比如cookie ，localstorage或sessionStorage 里面

**注意：**

这个秘钥是存储在服务端的，服务端验证的时候会先 通过base64逆向得到token的前两部分，也就是header 和payload，这样就得到了header声明的加密算法，然后再通过此加密算法和秘钥 对header和payload进行加盐 得到新的签名，跟token的第三部分签名对比，一样表示验证通过，不一样表示验证失败。



## 3. springboot 使用jwt

### 3.1 在pom.xml添加maven依赖

```xml
 <!-- JWT相关 -->
            <dependency>
                <groupId>io.jsonwebtoken</groupId>
                <artifactId>jjwt</artifactId>
                <version>0.7.0</version>
            </dependency>
```

### 3.2 创建jwt 工具类

具体根据自己实际情况写，大概就是一个生成token的方法和一个校验token的方法

```java 
package net.xdclass.xdvideo.utils;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import net.xdclass.xdvideo.domain.User;

import java.util.Date;

/**
 * jwt工具类
 */
public class JwtUtils {


    public static final String SUBJECT = "xdclass";

    public static final long EXPIRE = 1000*60*60*24*7;  //过期时间，毫秒，一周

    //秘钥
    public static final  String APPSECRET = "af213sdlkfa32";

    /**
     * 生成jwt
     * @param user
     * @return
     */
    public static String geneJsonWebToken(User user){

        if(user == null || user.getId() == null || user.getName() == null
                || user.getHeadImg()==null){
            return null;
        }
        String token = Jwts.builder().setSubject(SUBJECT)
                .claim("id",user.getId())
                .claim("name",user.getName())
                .claim("img",user.getHeadImg())
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis()+EXPIRE))
                .signWith(SignatureAlgorithm.HS256,APPSECRET).compact();

        return token;
    }


    /**
     * 校验token
     * @param token
     * @return
     */
    public static Claims checkJWT(String token ){

        try{
            final Claims claims =  Jwts.parser().setSigningKey(APPSECRET).
                    parseClaimsJws(token).getBody();
            return  claims;

        }catch (Exception e){ }
        return null;

    }

}

```

这里只是简单了解下jwt 和几个常见的验证方式，可以参考其他博客。比如

[玩转springboot2.x之整合JWT篇](https://blog.csdn.net/ljk126wy/article/details/82751787)

[[3种web会话管理的方式](https://www.cnblogs.com/lyzg/p/6067766.html)](http://www.cnblogs.com/lyzg/p/6067766.html)

[[看图理解JWT如何用于单点登录](https://www.cnblogs.com/lyzg/p/6132801.html)](https://www.cnblogs.com/lyzg/p/6132801.html)

[优化单点登录流程的好东西：JWT 介绍](https://cloud.tencent.com/developer/article/1113910)

网上很多资料，总之jwt是一个身份验证并且防篡改的一套机制，怎么用，看情况。至于我们要实现一套单点登录系统，可能用jwt 并没有使它更简单，也许传统方式也不错。具体我们不做深究，比如我们可以用CAS、jwt、redis等等，根据业务选择适合自己的方案，网上也有不少方案，没有最好，只有最合适。







