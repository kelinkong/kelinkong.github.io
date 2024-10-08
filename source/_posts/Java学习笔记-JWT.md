---
title: Java学习笔记-JWT
date: 2024-10-08 10:00:10
categories: [Java]
---
## 前言
在Java实战项目中，对于登陆操作，想要达到下面的效果：
- 登陆成功后，登陆的状态保持一段时间，不需要重复登陆

实现登陆保持功能可以使用session和cookie，但是这种方式有一些问题：
- session和cookie是存储在服务端的，如果服务端重启，session和cookie会丢失
- 对于分布式系统，session和cookie需要做共享，增加了复杂度

所以在该项目中，我使用JWT做登陆保持。

## JWT
### 概念
JWT（JSON Web Token）是一种基于JSON的开放标准（RFC 7519），用于在网络上传输声明的一种方式。JWT是一种轻量级的身份验证和授权的方式，可以在用户和服务器之间传递安全可靠的信息。

### 原理
JWT 的原理是，服务器认证以后，生成一个 JSON 对象，发回给用户，就像下面这样。

```json
{
  "姓名": "张三",
  "角色": "管理员",
  "到期时间": "2018年7月1日0点0分"
}
```
以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名（详见后文）。

服务器就不保存任何 session 数据了，也就是说，服务器变成无状态了，从而比较容易实现扩展。

### 缺点
- 一旦 JWT 签发了，就不能撤回，除非改密钥。（在到期时间之前，都是有效的）
- 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输

## 实现

### 基本流程
- 用户登录时，服务器验证用户身份。
- 如果验证通过，服务器生成一个包含用户信息的 JWT 并返回给客户端。
- 客户端将 JWT 存储在本地存储或 Cookie 中，并在后续请求的 Authorization 头中携带该 Token。(可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息Authorization字段里面。)
- 服务器每次根据 Token 验证用户身份，无需存储任何会话信息。

### 前端需要做什么？

前端需要在用户登录成功后，将服务器返回的 JWT 存储在本地存储或 Cookie 中，并在后续请求的 Authorization 头中携带该 Token。

用户登陆成功后，将服务器返回的 Token 存储在本地存储中。
```javascript
localStorage.setItem('token', token);
```

后续请求时，需要在请求头携带token。

如果每一个请求都手动去携带 Token，会很麻烦，所以可以使用 Axios 拦截器来实现。

```javascript
const instance = axios.create({
    baseURL: API_BASE_URL,
    headers: {
        'Content-Type': 'application/json'
    }
});

instance.interceptors.request.use(
    config => {
        const token = localStorage.getItem('token');
        if (token) {
            config.headers['Authorization'] = `Bearer ${token}`;
        }
        return config;
    },
    error => {
        return Promise.reject(error);
    }
);
```

### 后端实现

后端需要实现以下功能：
- JWT工具类，用于生成和解析（验证）Token
- 过滤器，用于过滤前端所有的请求，验证Token的有效性
- Web配置，将过滤器注册到Spring容器中

#### 依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

#### JWT工具类

生成和解析Token的工具类，使用了 jjwt 库。

```java
public class JwtUtil {

    private static final String KEY = "";//设置密钥（要想非对称加密这里换成私钥）

    //接收业务数据,生成token并返回
    public static String generateToken(Map<String, Object> claims) {
        return JWT.create()
                .withClaim("claims", claims) //token中加入用户信息
                .withExpiresAt(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 12)) //设置超时时间
                .sign(Algorithm.HMAC256(KEY)); //设置加密类型及密钥
    }

    //接收token,验证token,并返回业务数据
    public static Map<String, Object> parseToken(String token) {
        //去除token的前缀标记"Bearer "
        var newToken = token.contains("Bearer ")?token.substring("Bearer ".length()):token;
        return JWT.require(Algorithm.HMAC256(KEY))
                .build()
                .verify(newToken)
                .getClaim("claims")
                .asMap();
    }
}
```
#### 过滤器

过滤器用于过滤前端所有的请求，验证Token的有效性。

创建一个认证对象，将用户信息放入认证对象中，然后将认证对象放入SecurityContextHolder中。

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    /*
        * This method is called by the filter chain to filter the request.
        * 所有的请求都会经过这个方法，我们可以在这里进行token的解析和用户的认证
     */
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String token = getTokenFromRequest(request);
        logger.info("Token: " + token);

        if (token != null) {
            try {
                Map<String, Object> claims = JwtUtil.parseToken(token);
                if (claims != null) {
                    UsernamePasswordAuthenticationToken authentication = getAuthentication(claims);
                    authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    SecurityContextHolder.getContext().setAuthentication(authentication);
                }
             } catch (TokenExpiredException e) {
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                response.getWriter().write("Token expired");
                return;
            } catch (Exception e) {
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                return;
            }
        }

        filterChain.doFilter(request, response);
    }

    /*
        * This method is used to extract the token from the request.
        * 从请求中提取token
     */
    private String getTokenFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }

    /*
        * This method is used to create an authentication object.
        * 创建一个认证对象
     */
    private UsernamePasswordAuthenticationToken getAuthentication(Map<String, Object> claims) {
        // Implement token parsing and authentication creation logic
        return new UsernamePasswordAuthenticationToken(claims, null, new ArrayList<>());
    }
}
```

#### Web配置

- 将过滤器注册到Spring容器中。
- 跨域配置。

```java
public class SecurityConfig {

    /**
     * Configures the security filter chain that carries out authentication and authorization.
     * @param http the HttpSecurity object to configure
     * @return the SecurityFilterChain object
     * @throws Exception if an error occurs during configuration
     */
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/api/user/login").permitAll()
                        .anyRequest().authenticated()
                )
                .cors(cors -> cors.configurationSource(corsConfigurationSource()))
                .addFilterBefore(new JwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(List.of("http://localhost:9000"));
        configuration.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(List.of("*"));
        configuration.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

## JWT结构示例
JWT 由三部分组成，分别是 Header、Payload 和 Signature，它们之间使用 . 分隔。

**示例**

1. Header：头部包含 JWT 的类型和使用的签名算法
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
经过 Base64Url 编码后的 Header 是：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```
2. Payload：它包含用户的声明信息，例如用户的 ID、用户名等。这些信息通常包括注册声明、自定义声明等
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
经过 Base64Url 编码后的 Payload 是：
```
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```
3. Signature：签名是对 Header 和 Payload 的签名，防止数据被篡改
```json
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```
经过 Base64Url 编码后的 Signature 是：
```
dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```
最终生成的 JWT 是：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```

解码后的结构：
```
Header: {
  "alg": "HS256",
  "typ": "JWT"
}
Payload: {
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
Signature: HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```