# 基于JWT的Token认证机制实现

## 什么是JWT

JSON Web Token(JWT) 是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。

## JWT组成

一个JWT实际上就是一个字符串，它有三部分组成，头部。载荷与签名。

- 头部(Header)

  头部用户描述关于该JWT的最基本的信息，例如其类型以及签名所用的算法等。这也可以被表示成一个JSON对象。

  ```json
  {"typ":"JWT","alg":"HS256"}
  ```

  在头部指明了签名算法是HS256算法。我们进行BASE64编码http://base64.xpcha.com/，编码后的字符串如下：

  ```txt
  eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
  ```

  小知识：Base64是一种基于64个可打印字符来表示二进制数据的表示方法。由于2的六次放等于64，所以每6个比特为一个单元，对应某个可打印字符。三个字节有24个比特，对应与4个Base64单元，即3个字节需要用4个可打印字符来表示。JDK中提供了非常方便的BASE64Encoder和BASE64Decoder，用它们可以非常方便的完成基于BASE64的编码和解码。

- 载荷(playload)

  载荷就是存放有效信息的地方。这个名字像是特指飞机承载的货品，这些有效信息包含三个部分：

  1.标准中注册的声明(建议但不强制使用)

  ```txt
  iss:jwt签发者
  sub:jwt所面向的用户
  aud:接收jwt的一方
  exp:jwt的过期时间,这个过期时间必须要大于签发时间
  nbf:定义在上面时间之前,该jwt都是不可用的
  iat:jwt的签发时间
  jti:jwt的唯一身份标识,主要用来为作为一次性token,从而回避重放攻击
  ```

  2.公共的声明

  公共的声明可以添加任何的信息,一般添加用户的相关信息或其他业务需要的必要信息,但不建议添加敏感信息,因为该不疯魔在客户端可解密。

  3.私有的声明

  私有声明是提供者和消费者所共同定义的声明,一般不建议存放敏感信息,因为base64是对称解密的，意味着该部分信息可以归类为铭文信息。

  这个指的就是自定义的claim。比如前面的那个结构举例重的admin和name都属于自定的claim。这些claim跟JWT标准规定的claim区别在于：JWT规定的claim,JWT的接收方在拿到JWT之后,都知道怎么对这些标志的claim进行验证(还不知道是否能够验证)。而private claims不会验证,除非明确告诉接收方要对这些claim进行验证以及规则才行。

  定义一个payload:

  ```json
  {"sub":"1234567890","name":"John Doe","admin":true}
  ```

  然后将其进行base64编码，得到JWT的第二部分

  ```txt
  eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
  ```

- 签证(signature)

   jwt的第三部分是一个签证信息,这个签证信息由三部分组成:

    ```txt
    head(base64后的)
    payload(base64后的)
    secret
    ```

    这个部分需要base64加密后的header和base64加密后的payload使用，连接组成的字符串，然后通过header重声明的加密方式进行加盐secret组合加密,然后就构成了jwt的第三部分。
   
   ```txt
   TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
   ```
   
   将三部分组成用连接成一个完整的字符串，构成了最终的JWT:
   
   ```txt
   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6I kpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7Hg Q
   ```
   
   注意：secret是保存在服务器端的，JWT的签发生成也是在服务器端的，secret就是用来进行JWT的签发和JWT的验证,所以它就是你的服务器段是私钥,在任何场景都不应该流露出去。一旦客户端得知这个secret,那就意味着客户端是可以自我签发JWT了。

## Java的JJWT实现JWT

- 什么事JJWT

  JJWT是一个提供端到端的JWT创建和验证的java库。永远免费和开源(Apache License,版本2.0),JJWT很容易使用和理解。它被设计成一个以建筑为重心的流畅界面，隐藏了它的大部分复杂性。

- JJWT快速入门

  - 创建maven工程,引入依赖

    ```xml
    <dependency>
    	<groupId>io.jsonwebtoken</groupId>
    	<artifactId>jjwt</artifactId>
    	<version>0.6.0</version>
    </dependency>
    ```

  - 创建类JwtTest，用于生成token

    ```java
        public static void main(String[] args) {
            JwtBuilder jwtBuilder = Jwts.builder()
                    .setId("111")
                    .setSubject("小熊")
                    .setIssuedAt(new Date())
                    .signWith(SignatureAlgorithm.HS256, "baerwang");
            System.out.println(jwtBuilder.compact());
        }
    ```

  - 测试运行，输出如下

    ```java
    eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIxMTEiLCJzdWIiOiLlsI_nhooiLCJpYXQiOjE1ODIzNjA3ODh9.tptgu2WcnESZkCgXX4reJIVbuwuWj6R7hscEuxz5AnM
    ```

    再次运行，会发现每次运行的结果是不一样的，因为我们的载荷中包含了时间。

  - token的解析

    我们刚才已经创建了token，在web应用中这个操作是由服务端进行然后发给客户端，客户端在下次向服务端发送请求时需要携带这个token(这就是好像拿一张门票一样)，那服务端接到这个token应该解析出token中的信息(例如用户id)，根据这些信息查询数据库返回相应的结果。

  - 创建ParseJwtTest

    ```java
     public static void main(String[] args) {
            String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIxMTEiLCJzdWIiOiLlsI_nhooiLCJpYXQiOjE1ODIzNjA3ODh9.tptgu2WcnESZkCgXX4reJIVbuwuWj6R7hscEuxz5AnM";
            Claims claims = Jwts.parser().setSigningKey("baerwang").parseClaimsJws(token).getBody();
            System.out.println(claims.getId());
            System.out.println(claims.getSubject());
            System.out.println(claims.getIssuedAt());
        }
    ```

    试着将token或签名密钥篡改一下，会发现运行时就会报错，所以解析token也就是验证token

  - 过期校验

    有很多时候，我们并不希望签发的token是永久生效的，所以我们可以为token添加一个过期时间。

  - 修改JwtTest

    ```java
        public static void main(String[] args) {
            /*当前时间 */
            long now = System.currentTimeMillis();
            /*过期时间为1分钟*/
            long exp = now + 1000*60;
            JwtBuilder jwtBuilder = Jwts.builder()
                    .setId("111")
                    .setSubject("小熊")
                    .setIssuedAt(new Date())
                    .signWith(SignatureAlgorithm.HS256, "baerwang")
                    /*当前时间+过期时间为1分钟*/
                    .setExpiration(new Date(exp));
            System.out.println(jwtBuilder.compact());
        }
    ```

  - 修改ParseJwtTest

    ```java
        public static void main(String[] args) {
            String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIxMTEiLCJzdWIiOiLlsI_nhooiLCJpYXQiOjE1ODIzNjE3OTksImV4cCI6MTU4MjM2MTg1OH0.EoBWXFXwcngJfyxY0l8p1dMKZRkE6kjK47BOEyVOubM";
            Claims claims = Jwts.parser().setSigningKey("baerwang").parseClaimsJws(token).getBody();
            System.out.println(claims.getId());
            System.out.println(claims.getSubject());
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy‐MM‐dd hh:mm:ss");
            System.out.println("签发时间:" + sdf.format(claims.getIssuedAt()));
            System.out.println("过期时间:" + sdf.format(claims.getExpiration()));
            System.out.println("当前时间:" + sdf.format(new Date()));
        }
    ```

    测试运行，当未过期时可以正常读取，当过期时会引发io.jsonwebtoken.ExpiredJwtException异常

    ```java
    Exception in thread "main" io.jsonwebtoken.ExpiredJwtException: JWT expired at 2020-02-22T16:57:38+0800. Current time: 2020-02-22T16:57:38+0800
    	at io.jsonwebtoken.impl.DefaultJwtParser.parse(DefaultJwtParser.java:365)
    	at io.jsonwebtoken.impl.DefaultJwtParser.parse(DefaultJwtParser.java:458)
    	at io.jsonwebtoken.impl.DefaultJwtParser.parseClaimsJws(DefaultJwtParser.java:518)
    	at com.baerwang.jwt.ParseJwtTest.main(ParseJwtTest.java:17)
    ```

  - 自定义claims，刚才的例子只是存储了id和subject两个信息，如果你想存储更多的信息(例如角色)可以定义自定义claims，创建JwtTest2

    ```java
        public static void main(String[] args) {
            /*当前时间 */
            long now = System.currentTimeMillis();
            /*过期时间为1分钟*/
            long exp = now + 1000 * 60;
            JwtBuilder jwtBuilder = Jwts.builder()
                    .setId("111")
                    .setSubject("小熊")
                    .setIssuedAt(new Date())
                    .signWith(SignatureAlgorithm.HS256, "baerwang")
                    /*当前时间+过期时间为1分钟*/
                    .setExpiration(new Date(exp))
                    .claim("roles", "admin")
                    .claim("logo", "logo.png");
            System.out.println(jwtBuilder.compact());
        }
    ```

  - 修改ParseJwtTest

    ```java
        public static void main(String[] args) {
            String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIxMTEiLCJzdWIiOiLlsI_nhooiLCJpYXQiOjE1ODIzNjIxMjcsImV4cCI6MTU4MjM2MjE4Niwicm9sZXMiOiJhZG1pbiIsImxvZ28iOiJsb2dvLnBuZyJ9.JqRlkQKsF058gbtPlU80CXxmwMOIrm-Ns4p118KlPz0";
            Claims claims = Jwts.parser().setSigningKey("baerwang").parseClaimsJws(token).getBody();
            System.out.println(claims.getId());
            System.out.println("subject:" + claims.getSubject());
            System.out.println("roles:" + claims.get("roles"));
            System.out.println("logo:" + claims.get("logo"));
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy‐MM‐dd hh:mm:ss");
            System.out.println("签发时间:" + sdf.format(claims.getIssuedAt()));
            System.out.println("过期时间:" + sdf.format(claims.getExpiration()));
            System.out.println("当前时间:" + sdf.format(new Date()));
        }
    ```

    
