---
title: JWT-token详解
date: 2021-08-31 23:10:03
tags: 
- token
categories:
- Java
description:
sticky:
cover: https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/undraw_3d_modeling_h60h.png
---

> 说到token，我们就要聊一聊基于token的认证和传统的session的区别。

## 传统的session认证

>  我们知道，http协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据http协议，我们并不能知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie,以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了,这就是传统的基于session认证。
>
>  但是这种基于session的认证使应用本身很难得到扩展，随着不同客户端用户的增加，独立的服务器已无法承载更多的用户，而这时候基于session认证应用的问题就会暴露出来。

## 基于session认证所显露的问题

> **Session**: 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大。
>
> **扩展性**: 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。
>
> **CSRF**: 因为是基于cookie来进行用户识别的, cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。

## 基于token的鉴权机制

> 基于token的鉴权机制类似于http协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。

流程上是这样的：

- 用户使用用户名密码来请求服务器
- 服务器进行验证用户的信息
- 服务器通过验证发送给用户一个token
- 客户端存储token，并在每次请求时附送上这个token值
- 服务端验证token值，并返回数据

这个token必须要在每次请求时传递给服务端，它应该保存在请求头里， 另外，服务端要支持`CORS(跨来源资源共享)`策略，一般我们在服务端这么做就可以了`Access-Control-Allow-Origin: *`。

那么我们现在回到JWT的主题上。

## JWT长什么样

> JWT是由三段信息构成的，将这三段信息文本用`.`链接一起就构成了Jwt字符串。就像这样:

~~~ token
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
~~~

## JWT的构成

> 一个JWT实际上就是一个字符串，它由三部分组成，头部、载荷与签证

第一部分我们称它为头部（header),第二部分我们称其为载荷（payload, 类似于飞机上承载的物品)，第三部分是签证（signature).

### header(头部)

> 头部用于描述关于该JWT的最基本的信息，例如其类型以及签名所用的算法等。这也可以被表示成一个JSON对象。

~~~json
{
  'typ': 'JWT',
  'alg': 'HS256'
}
~~~

在头部指明了签名算法是HS256算法。 我们进行[BASE64编
码](http://base64.xpcha.com/)，编码后的字符串如下：

~~~token
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
~~~

> 小知识：Base64是一种基于64个可打印字符来表示二进制数据的表示方法。由于2的6次方等于64，所以每6个比特为一个单元，对应某个可打印字符。三个字节有24个比特，对应于4个Base64单元，即3个字节需要用4个可打印字符来表示。JDK 中提供了非常方便的 BASE64Encoder 和 BASE64Decoder，用它们可以非常方便的完成基于 BASE64 的编码和解码

### playload(载荷)

载荷就是存放有效信息的地方。这个名字像是特指飞机上承载的货品，这些有效信息包含三个部分

- 标准中注册的声明
- 公共的声明
- 私有的声明

**标准中注册的声明** (建议但不强制使用) ：

- **iss**: jwt签发者
- **sub**: jwt所面向的用户
- **aud**: 接收jwt的一方
- **exp**: jwt的过期时间，这个过期时间必须要大于签发时间
- **nbf**: 定义在什么时间之前，该jwt都是不可用的.
- **iat**: jwt的签发时间
- **jti**: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。

**公共的声明** ：
 公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密.

**私有的声明** ：
 私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

定义一个payload:

~~~json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
~~~

然后将其进行base64加密，得到Jwt的第二部分。

```token
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```

### signature(签证)

jwt的第三部分是一个签证信息，这个签证信息由三部分组成：

- header (base64后的)
- payload (base64后的)
- secret

这个部分需要base64加密后的header和base64加密后的payload使用`.`连接组成的字符串，然后通过header中声明的加密方式进行加盐`secret`组合加密，然后就构成了jwt的第三部分。在实际的应用场景中，`secret`通常使用用户的密码

```token
TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

将这三部分用`.`连接成一个完整的字符串,构成了最终的jwt:

```token
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

**注意：secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。**

## 如何应用

一般是在请求头里加入`Authorization`，并加上`Bearer`标注：

~~~js
fetch('api/user/1', {
  headers: {
    'Authorization': 'Bearer ' + token
  }
})
~~~

服务端会验证token，如果验证通过就会返回相应的资源。整个流程就是这样的:

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/1821058-2e28fe6c997a60c9.png)

## Java的JJWT实现JWT

### 什么是JJWT

JJWT是一个JWT创建和验证的Java库。

### token的创建

- 引入依赖

~~~xml
<dependency>
	<groupId>io.jsonwebtoken</groupId>
	<artifactId>jjwt</artifactId>
	<version>0.6.0</version>
</dependency>
~~~

- 创建类CreatejwtTest，用于生成token

~~~java
public class CreateJwtTest {
	public static void main(String[] args) {
		JwtBuilder builder= Jwts.builder().setId("888")
			.setSubject("小白")
			.setIssuedAt(new Date())//用于设置签发时间
			.signWith(SignatureAlgorithm.HS256,"wangmh");//用于设置签名秘钥
		System.out.println( builder.compact() );
	}
}
~~~

- 测试

~~~
eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1MjM0M
TM0NTh9.gq0J‐cOM_qCNqU_s‐d_IrRytaNenesPmqAIhQpYXHZk
#再次运行，每次运行结果都会不一样，因为我们载荷中包含了时间
~~~

### token的解析

~~~java
public class ParseJwtTest {
	public static void main(String[] args) {
		String compactJws="eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1MjM0MTM0NTh9.gq0J‐cOM_qCNqU_s‐d_IrRytaNenesPmqAIhQpYXHZk";
		Claims claims =
		Jwts.parser().setSigningKey("wangmh").parseClaimsJws(compactJws).getBody();
		System.out.println("id:"+claims.getId());
		System.out.println("subject:"+claims.getSubject());
		System.out.println("IssuedAt:"+claims.getIssuedAt());
	}
}
//试着将token或签名秘钥篡改一下，会发现运行时就会报错，所以解析token也就是验证token
~~~

### token过期校验

有很多时候，我们并不希望签发的token是永久生效的，所以我们可以为token添加一个过期时间。

~~~java
public class CreateJwtTest2 {
	public static void main(String[] args) {
	//为了方便测试，我们将过期时间设置为1分钟
	long now = System.currentTimeMillis();//当前时间
	long exp = now + 1000*60;//过期时间为1分钟
	JwtBuilder builder= Jwts.builder().setId("888")
        .setSubject("小白")
        .setIssuedAt(new Date())
        .signWith(SignatureAlgorithm.HS256,"wangmh")
        .setExpiration(new Date(exp));//用于设置过期时间
	System.out.println( builder.compact() );
	}
}

~~~

~~~java
public class ParseJwtTest {
	public static void main(String[] args) {
	String compactJws="eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1MjM0MTY1NjksImV4cCI6MTUyMzQxNjYyOX0.Tk91b6mvyjpKcldkic8DgXz0zsPFFnRgTgkgcAsa9cc";
	Claims claims =Jwts.parser().setSigningKey("wangmh").parseClaimsJws(compactJws).getBody();
	System.out.println("id:"+claims.getId());
	System.out.println("subject:"+claims.getSubject());
	SimpleDateFormat sdf=new SimpleDateFormat("yyyy‐MM‐dd hh:mm:ss");
	System.out.println("签发时间:"+sdf.format(claims.getIssuedAt()));
	System.out.println("过期时间:"+sdf.format(claims.getExpiration()));
	System.out.println("当前时间:"+sdf.format(new Date()) );
	}
}

//测试运行，当未过期时可以正常读取，当过期时会引发io.jsonwebtoken.ExpiredJwtException异常。

~~~

### 自定义claims

我们刚才的例子只是存储了id和subject两个信息，如果你想存储更多的信息（例如角色）可以定义自定义claims

~~~java
public class CreateJwtTest3 {
	public static void main(String[] args) {
		//为了方便测试，我们将过期时间设置为1分钟
		long now = System.currentTimeMillis();//当前时间
		long exp = now + 1000*60;//过期时间为1分钟
		JwtBuilder builder= Jwts.builder().setId("888")
                                .setSubject("小白")
                                .setIssuedAt(new Date())
                                .signWith(SignatureAlgorithm.HS256,"wangmh")
                                .setExpiration(new Date(exp))
                                .claim("roles","admin")
                                .claim("logo","logo.png");
		System.out.println( builder.compact() );
        
        
        String compactJwt = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1NTQ5NDcxNTksImV4cCI6MTU1NDk0NzIxOSwicm9sZXMiOiJhZG1pbiIsImxvZ28iOiJsb2dvLnBuZyJ9.HD_myvdNjOGGnu4p8Z8QX9dnXHJEZa0nsLKxOFRiYJY";
        Claims claims=Jwts.parser().setSigningKey("wangmh").parseClaimsJws(compactJwt).getBody();
        System.out.println("id:"+claims.getId());
        System.out.println("subject:"+claims.getSubject());
        System.out.println("roles:"+claims.get("roles"));
        System.out.println("logo:"+claims.get("logo"));
        SimpleDateFormat sdf=new SimpleDateFormat("yyyy‐MM‐dd hh:mm:ss");
        System.out.println("签发时间:"+sdf.format(claims.getIssuedAt()));
        System.out.println("过期时间:"+sdf.format(claims.getExpiration()));
        System.out.println("当前时间:"+sdf.format(new Date()) );
	}
}
~~~

### 案例实现

~~~java
//JwtUtil.java
@ConfigurationProperties("jwt.config")
public class JwtUtil {
    private String key ;
    private long ttl ;//一个小时
    public String getKey() {
        return key;
    }
    public void setKey(String key) {
        this.key = key;
    }
    public long getTtl() {
        return ttl;
    }

    public void setTtl(long ttl) {
        this.ttl = ttl;
    }

    /**
     * 生成JWT
     *
     * @param id
     * @param subject
     * @return
     */
    public String createJWT(String id, String subject, String roles) {
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        JwtBuilder builder = Jwts.builder().setId(id)
                .setSubject(subject)
                .setIssuedAt(now)
                .signWith(SignatureAlgorithm.HS256, key).claim("roles", roles);
        if (ttl > 0) {
            builder.setExpiration( new Date( nowMillis + ttl));
        }
        return builder.compact();
    }

    /**
     * 解析JWT
     * @param jwtStr
     * @return
     */
    public Claims parseJWT(String jwtStr){
        return  Jwts.parser()
                .setSigningKey(key)
                .parseClaimsJws(jwtStr)
                .getBody();
    }
}
~~~

~~~yml
#配置文件
jwt:
   config:
        key: wangmh
        ttl: 3600000
~~~

#### 登录鉴权

~~~java
//Controller.java
@Autowired
private JwtUtil jwtUtil;

@RequestMapping(value="/login",method=RequestMethod.POST)
public Result login(@RequestBody Map<String,String> loginMap){
	Admin admin =adminService.findByLoginnameAndPassword(loginMap.get("loginname"),loginMap.get("password"));
	if(admin!=null){
		//生成token
		String token = jwtUtil.createJWT(admin.getId(),admin.getLoginname(), "admin");
		Map map=new HashMap();
		map.put("token",token);
		map.put("name",admin.getLoginname());//登陆名
		return new Result(true,StatusCode.OK,"登陆成功",map);
	}else{
		return new Result(false,StatusCode.LOGINERROR,"用户名或密码错误");
	}
}
~~~

#### **删除用户功能鉴权**

~~~java
@Autowired
private HttpServletRequest request;
/**
* 删除
* @param id
*/
@RequestMapping(value="/{id}",method= RequestMethod.DELETE)
public Result delete(@PathVariable String id ){
	String authHeader = request.getHeader("Authorization");//获取头信息
	if(authHeader==null){
		return new Result(false,StatusCode.ACCESSERROR,"权限不足");
	} 
   	if(!authHeader.startsWith("Bearer ")){
		return new Result(false,StatusCode.ACCESSERROR,"权限不足");
	} 
    String token=authHeader.substring(7);//提取token
	Claims claims = jwtUtil.parseJWT(token);
	if(claims==null){
		return new Result(false,StatusCode.ACCESSERROR,"权限不足");
	} 
    if(!"admin".equals(claims.get("roles"))){
		return new Result(false,StatusCode.ACCESSERROR,"权限不足");
	} 
    userService.deleteById(id);
	return new Result(true,StatusCode.OK,"删除成功");
}
~~~

#### **使用拦截器方式实现token鉴权**

> 如果我们每个方法都去写一段代码，冗余度太高，不利于维护。因此我们可以使用拦截器的方式去实现token鉴权

**添加拦截器**

> Spring为我们提供了org.springframework.web.servlet.handler.HandlerInterceptorAdapter这个适配器，继承此类，可以非常方便的实现自己的拦截器。他有三个方法：分别实现预处理、后处理（调用了Service并返回ModelAndView，但未进行页面渲染）、返回处理（已经渲染了页面）在preHandle中，可以进行编码、安全控制等处理；
> 在postHandle中，有机会修改ModelAndView；
> 在afterCompletion中，可以根据ex是否为null判断是否发生了异常，进行日志记录

- 创建拦截器类

~~~java
//JwtFilter.java
@Component
public class JwtFilter extends HandlerInterceptorAdapter {
	@Autowired
	private JwtUtil jwtUtil;
	@Override
	public boolean preHandle(HttpServletRequest request,HttpServletResponse response, Object handler)throws Exception {
		System.out.println("经过了拦截器");
        final String authHeader = request.getHeader("Authorization");
		if (authHeader != null && authHeader.startsWith("Bearer ")) {
			final String token = authHeader.substring(7); // The partafter "Bearer "
			Claims claims = jwtUtil.parseJWT(token);
			if (claims != null) {
				if("admin".equals(claims.get("roles"))){//如果是管理员
					request.setAttribute("admin_claims", claims);
				} 
                if("user".equals(claims.get("roles"))){//如果是用户
					request.setAttribute("user_claims", claims);
				}
			}
		}
		return true;
	}
}
~~~

- 配置拦截器类

~~~java
@Configuration
public class ApplicationConfig extends WebMvcConfigurationSupport {
	@Autowired
	private JwtFilter jwtFilter;
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(jwtFilter).
			addPathPatterns("/**").
			excludePathPatterns("/**/login");
	}
}
~~~

- 删除功能实现

~~~java
/**
* 删除
* @param id
*/
@RequestMapping(value="/{id}",method= RequestMethod.DELETE)
public Result delete(@PathVariable String id ){
	Claims claims=(Claims) request.getAttribute("admin_claims");
	if(claims==null){
		return new Result(true,StatusCode.ACCESSRROR,"无权访问");
	} 
	userService.deleteById(id);
	return new Result(true,StatusCode.OK,"删除成功");
}
~~~

## 对Token认证的五点认识

- 一个Token就是一些信息的集合；
- 在Token中包含足够多的信息，以便在后续请求中减少查询数据库的几率；
- 服务端需要对cookie和HTTP Authrorization Header进行Token信息的检查；
- 基于上一点，你可以用一套token认证代码来面对浏览器类客户端和非浏览器类客户端；
- 因为token是被签名的，所以我们可以认为一个可以解码认证通过的token是由我们系统发放的，其中带的信息是合法有效的；

## Token的优点 

- **支持跨域访问**: Cookie是不允许垮域访问的，这一点对Token机制是不存在的，前提是传输的用户认证信息通过HTTP头传输。
- **无状态(也称：服务端可扩展行)**:Token机制在服务端不需要存储session信息，因为Token 自身包含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息。
- **更适用CDN**: 可以通过内容分发网络请求你服务端的所有资料（如：javascript，HTML,图片等），而你的服务端只要提供API即可。
- **去耦**: 不需要绑定到一个特定的身份验证方案。Token可以在任何地方生成，只要在你的API被调用的时候，你可以进行Token生成调用即可。
- **更适用于移动应用**: 当你的客户端是一个原生平台（iOS, Android，Windows 8等）时，Cookie是不被支持的（你需要通过Cookie容器进行处理），这时采用Token认证机制就会简单得多。
- **CSRF**:因为不再依赖于Cookie，所以你就不需要考虑对CSRF（跨站请求伪造）的防范。
- **性能**: 一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算 的Token验证和解析要费时得多。
- **不需要为登录页面做特殊处理**: 如果你使用Protractor 做功能测试的时候，不再需要为登录页面做特殊处理。
- **基于标准化**:你的API可以采用标准化的 JSON Web Token (JWT). 这个标准已经存在多个后端库（.NET, Ruby, Java,Python, PHP）和多家公司的支持（如：Firebase,Google, Microsoft）。
- 因为json的通用性，所以JWT是可以进行跨语言支持的，像JAVA,JavaScript,NodeJS,PHP等很多语言都可以使用。
- 因为有了payload部分，所以JWT可以在自身存储一些其他业务逻辑所必要的非敏感信息。
- 便于传输，jwt的构成非常简单，字节占用很小，所以它是非常便于传输的。
- 它不需要在服务端保存会话信息, 所以它易于应用的扩展。

## 安全相关 

- 不应该在jwt的payload部分存放敏感信息，因为该部分是客户端可解密的部分。
- 保护好secret私钥，该私钥非常重要。
- 如果可以，请使用https协议

