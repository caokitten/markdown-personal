### 登录认证

登录校验: 服务器接收浏览器的请求, 先校验用户是否登录

- 会话跟踪技术:
  - Cookie: 数据存储在浏览器当中
  - Session: 数据存储在服务端
  - 令牌技术

#### Cookie 方案

- 响应头: 设置 Cookie 数据

- 请求头: 携带 Cookie 数据

- 服务器自动响应给浏览器 Cookie

  ‘服务器只是发出指令，告诉浏览器要存储什么 Cookie，但服务器自己不会保存这些 Cookie 的值 服务器完全信任客户端发来的 Cookie 值’

- Cookie 校验:

  - 服务器生成 Cookie 时，对原始数据计算 **签名**（如 HMAC-SHA256）
  - 发送拼接后的字符串给浏览器
  - 浏览器下次请求时带回该值
  - 服务器拆分值并 **使用只有服务器知道的密钥计算签名进行比对**

```java
@Slf4j
@RestController
public class CookieController {

    //设置Cookie
    @GetMapping("/c1")
    public Result cookie1(HttpServletResponse response){
        response.addCookie(new Cookie("login_username","itheima")); //设置Cookie/响应Cookie
        return Result.success();
    }
        
    //获取Cookie
    @GetMapping("/c2")
    public Result cookie2(HttpServletRequest request){
        Cookie[] cookies = request.getCookies();
        for (Cookie cookie : cookies) {
            if(cookie.getName().equals("login_username")){
                System.out.println("login_username: "+cookie.getValue()); //输出name为login_username的cookie
            }
        }
        return Result.success();
    }
}    
//缺点:移动端无法使用
//Cookie无法跨域
```

#### 跨域问题

一般情况下, 前后端分开部署, 前端服务器(Nginx)与后端服务器在不同 ip 地址上, 即 html 页面与后端资源在不同的服务器上

满足一下三者任一不同都为跨域:

- 协议
- IP
- 端口

#### Session 方案

底层基于 Cookie 实现, SessionID 的生成不依赖于任何用户输入的数据

- 浏览器第一次请求服务器, 服务器获取会话对象 Session, 自动创建一个 Session 对象, 生成对应 SessionID
- 服务器将 SessionID 响应给浏览器, 即在 Cookie 中添加一个响应头, 浏览器识别并存储该 Cookie

```java
@Slf4j
@RestController
public class SessionController {

    @GetMapping("/s1")
    public Result session1(HttpSession session){
        log.info("HttpSession-s1: {}", session.hashCode());

        session.setAttribute("loginUser", "tom"); //往session中存储数据
        return Result.success();
    }

    @GetMapping("/s2")
    public Result session2(HttpServletRequest request){
        HttpSession session = request.getSession();
        log.info("HttpSession-s2: {}", session.hashCode());

        Object loginUser = session.getAttribute("loginUser"); //从session中获取数据
        log.info("loginUser: {}", loginUser);
        return Result.success(loginUser);
    }
}
//缺点:服务器集群环境下无法使用Session
//依赖于Cookie,与Cookie缺点一致,但比Cookie更安全
```

#### 服务器集群环境问题

项目一般不会只部署在一台服务器上, 以保证一台服务器出错时, 该应用仍可正常访问

- 负载均衡服务器

  当用户访问该网站时, 会先访问一台前置服务器, 即负载均衡服务器, 将前端发起的请求均匀地分发给其他服务器

- 由于 sessionID 在一个服务器上存储, 若本次请求发送给其他服务器上, 则无法查询到该 SessionID, 致使出错

#### 令牌技术方案

- 本质是一个字符串, 在用户登录成功后生成, 是用户的合法身份凭证, 响应数据时直接将数据响应给前端

- 前端接收到令牌后将令牌存储起来, 即可存储在 Cookie 中, 也可存储在其他的存储空间

- 后续每一次请求都需要将令牌携带到服务端, 服务端校验令牌的有效性

  ```json
  // JWT 令牌示例
  {
    "userId": 123,
    "role": "admin",
    "lastAction": "2023-08-15T10:00:00Z",
    "signature": "x7Y9a..." // 防篡改签名
  }
  ```

- 缺点: 需要自己实现（包括令牌的生成、令牌的传递、令牌的校验）

##### JWT 令牌

即将原始的 json 格式数据进行安全封装

组成:

- Header: 头, 记录令牌类型, 签名算法等
- Payload: 有效载荷, 携带自定义信息及默认信息
- Signature: 签名, 防止 Token 被篡改

JSON 格式转字符串:

​	使用 Base64 编码: 基于 64 个可打印的字符(A-Z, a-z,0-9,+,/, 补位:=)对二进制数据进行的编码

生成 JWT 令牌:

```xml
<!-- JWT依赖-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

```java
@Test
public void testGenJwt() {
    Map<String, Object> claims = new HashMap<>();
    claims.put("id", 10);
    claims.put("username", "itheima");

    String jwt = Jwts.builder().signWith(SignatureAlgorithm.HS256, "aXRjYXN0")
        .addClaims(claims)
        .setExpiration(new Date(System.currentTimeMillis() + 12 * 3600 * 1000))
        .compact();
	//Jwts.builder()创建JWT构建器对象
    //.signWith(SignatureAlgorithm.HS256, "aXRjYXN0")指定签名算法及密钥
    //.addClaims(claims)添加自定义数据,形参要求为Map类型
    //.setExpiration(new Date(System.currentTimeMillis() + 12 * 3600 * 1000))设置令牌有效期
    //.compact();构建令牌
    System.out.println(jwt);
}

//令牌校验
@Test
public void testParseJwt() {
    Claims claims = Jwts.parser().setSigningKey("aXRjYXN0")
        .parseClaimsJws("eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MTAsInVzZXJuYW1lIjoiaXRoZWltYSIsImV4cCI6MTcwMTkwOTAxNX0.N-MD6DmoeIIY5lB5z73UFLN9u7veppx1K5_N_jS9Yko")
        .getBody();
    //Jwts.parser()令牌解析器对象
    //.setSigningKey("aXRjYXN0")提供密钥
    //.parseClaimsJws("eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MTAsInVzZXJuYW1lIjoiaXRoZWltYSIsImV4cCI6MTcwMTkwOTAxNX0.N-MD6DmoeIIY5lB5z73UFLN9u7veppx1K5_N_jS9Yko")提供令牌
    //.getBody();获取自定义数据部分
    System.out.println(claims);
}
```

##### 过滤器 Filter

将对资源的请求拦截下来, 实现一些特殊的功能, 一般用于: 登录校验, 统一编码处理, 敏感字符处理

- 定义 Filter

  ```java
  public class DemoFilter implements Filter {
      //初始化方法, web服务器启动, 创建Filter实例时调用, 只调用一次
      public void init(FilterConfig filterConfig) throws ServletException {
          System.out.println("init ...");
      }
  
      //拦截到请求时,调用该方法,可以调用多次
      public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {
          System.out.println("拦截到了请求...");
      }
  
      //销毁方法, web服务器关闭时调用, 只调用一次
      public void destroy() {
          System.out.println("destroy ... ");
      }
  }
  ```

- 配置 Filter

  ```java
  //在定义完Filter之后，Filter其实并不会生效，还需要完成Filter的配置，Filter的配置非常简单，只需要在Filter类上添加一个注解：@WebFilter，并指定属性urlPatterns，通过这个属性指定过滤器要拦截哪些请求
  @WebFilter(urlPatterns = "/*") //配置过滤器要拦截的请求路径（ /* 表示拦截浏览器的所有请求 ）
  public class DemoFilter implements Filter {
      //初始化方法, web服务器启动, 创建Filter实例时调用, 只调用一次,可选
      public void init(FilterConfig filterConfig) throws ServletException {
          System.out.println("init ...");
      }
  
      //拦截到请求时,调用该方法,可以调用多次,必须重写
      public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {
          System.out.println("拦截到了请求...");
      }
  
      //销毁方法, web服务器关闭时调用, 只调用一次,可选
      public void destroy() {
          System.out.println("destroy ... ");
      }
  }
  ```

  当我们在 Filter 类上面加了@WebFilter 注解之后，接下来我们还需要在启动类上面加上一个注解 `@ServletComponentScan`，通过这个 `@ServletComponentScan` 注解来开启 SpringBoot 项目对于 Servlet 组件的支持:

  ```java
  @ServletComponentScan //开启对Servlet组件的支持
  @SpringBootApplication
  public class TliasManagementApplication {
      public static void main(String[] args) {
          SpringApplication.run(TliasManagementApplication.class, args);
      }
  }
  ```

  ```java
  //在过滤器Filter中，如果不执行放行操作，将无法访问后面的资源。 放行操作：
  chain.doFilter(request, response);
  ```

  并非所有的请求都需要校验令牌, 在拦截到请求后, 登录请求不需要验证令牌, 当令牌合法后通过放行:
  ![image](./day5.assets/image.png)

实现:

```java
package com.itheima.filter;

import com.itheima.utils.JwtUtils;
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpStatus;
import org.springframework.util.StringUtils;
import java.io.IOException;

/**
 * 令牌校验过滤器
 */
@Slf4j
@WebFilter(urlPatterns = "/*")
public class TokenFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;
        //1. 获取请求url。
        String url = request.getRequestURL().toString();

        //2. 判断请求url中是否包含login，如果包含，说明是登录操作，放行。
        if(url.contains("login")){ //登录请求
            log.info("登录请求 , 直接放行");
            chain.doFilter(request, response); // 放行
            return;
        }

        //3. 获取请求头中的令牌（token）。
        String jwt = request.getHeader("token");

        //4. 判断令牌是否存在，如果不存在，返回错误结果（未登录）。
        if(!StringUtils.hasLength(jwt)){ //jwt为空
            log.info("获取到jwt令牌为空, 返回错误结果");
            response.setStatus(HttpStatus.SC_UNAUTHORIZED); //响应错误信息
            return;
        }

        //5. 解析token，如果解析失败，返回错误结果（未登录）。
        try {
            JwtUtils.parseJWT(jwt);
        } catch (Exception e) {
            e.printStackTrace();
            log.info("解析令牌失败, 返回错误结果");
            response.setStatus(HttpStatus.SC_UNAUTHORIZED); // 同上
            return;
        }

        //6. 放行。
        log.info("令牌合法, 放行");
        chain.doFilter(request , response);
    }

}
```

执行流程:

- 请求拦截--> 放行前--> 放行--> 资源访问--> 响应拦截--> 放行后

- doFilter 之前的方法都属于放行前, doFilter 中执行放行操作, 资源访问完成后返回经过过滤器, 此为放行后, 在 doFilter 中执行

- eg:

  ```java
  @WebFilter(urlPatterns = "/*") 
  public class DemoFilter implements Filter {
      
      @Override //初始化方法, 只调用一次
      public void init(FilterConfig filterConfig) throws ServletException {
          System.out.println("init 初始化方法执行了");
      }
      
      @Override
      public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
          
          System.out.println("DemoFilter   放行前逻辑.....");
  
          //放行请求
          filterChain.doFilter(servletRequest,servletResponse);
  
          System.out.println("DemoFilter   放行后逻辑.....");
          
      }
  
      @Override //销毁方法, 只调用一次
      public void destroy() {
          System.out.println("destroy 销毁方法执行了");
      }
  }
  ```

拦截路径 urlPatterns:

- | 拦截路径     | urlPatterns 值 | 含义                               |
  | ------------ | ------------- | ---------------------------------- |
  | 拦截具体路径 | /login        | 只有访问 /login 路径时，才会被拦截 |
  | 目录拦截     | /emps/*       | 访问/emps 下的所有资源，都会被拦截  |
  | 拦截所有     | /*            | 访问所有资源，都会被拦截           |

过滤器链: 一个 web 程序中可以配置多个过滤器, 形成过滤器链

- **过滤器的执行顺序优先级按照过滤器类名(字符串)的自然排序进行**

##### 拦截器 Interceptor

- 一种动态拦截方法调用的机制, 类似于过滤器
- 由 Spring 框架提供
- 拦截请求, 在指定方法调用前后执行设定的业务逻辑

基本使用:

- 定义拦截器:

  ```java
  //自定义拦截器
  @Component
  public class DemoInterceptor implements HandlerInterceptor {
      //preHandle:目标资源方法执行前执行。 返回true：放行    返回false：不放行
      @Override
      public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
          System.out.println("preHandle .... ");
          
          return true; //true表示放行
      }
  
      //postHandle:目标资源方法执行后执行
      @Override
      public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
          System.out.println("postHandle ... ");
      }
  
      //afterCompletion:视图渲染完毕后执行，最后执行
      @Override
      public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
          System.out.println("afterCompletion .... ");
      }
  }
  ```

- 注册拦截器

  ```java
  //Application同级目录下创建包,包内创建配置类WebConfig,实现WebMvcConfigurer接口,重写addInterceptors方法
  @Configuration  
  public class WebConfig implements WebMvcConfigurer {
  
      //自定义的拦截器对象
      @Autowired
      private DemoInterceptor demoInterceptor;
  
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
         //注册自定义拦截器对象
          registry.addInterceptor(demoInterceptor).addPathPatterns("/**");//设置拦截器拦截的请求路径（ /** 表示拦截所有请求）
      }
  }
  ```

通过拦截器实现令牌校验:

- ```java
  @Slf4j
  @Component
  public class TokenInterceptor implements HandlerInterceptor {
      @Override
      public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
          //1. 获取请求url。
          String url = request.getRequestURL().toString();
  
          //2. 判断请求url中是否包含login，如果包含，说明是登录操作，放行。
          if(url.contains("login")){ //登录请求
              log.info("登录请求 , 直接放行");
              return true;
          }
  
          //3. 获取请求头中的令牌（token）。
          String jwt = request.getHeader("token");
  
          //4. 判断令牌是否存在，如果不存在，返回错误结果（未登录）。
          if(!StringUtils.hasLength(jwt)){ //jwt为空
              log.info("获取到jwt令牌为空, 返回错误结果");
              response.setStatus(HttpStatus.SC_UNAUTHORIZED);
              return false;
          }
  
          //5. 解析token，如果解析失败，返回错误结果（未登录）。
          try {
              JwtUtils.parseJWT(jwt);
          } catch (Exception e) {
              e.printStackTrace();
              log.info("解析令牌失败, 返回错误结果");
              response.setStatus(HttpStatus.SC_UNAUTHORIZED);
              return false;
          }
  
          //6. 放行。
          log.info("令牌合法, 放行");
          return true;
      }
  
  }
  ```

- ```java
  @Configuration  
  public class WebConfig implements WebMvcConfigurer {
      //拦截器对象
      @Autowired
      private TokenInterceptor tokenInterceptor;
  
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
         //注册自定义拦截器对象
          registry.addInterceptor(tokenInterceptor).addPathPatterns("/**");
      }
  }
  
  ```

拦截路径配置:

```java
registry.addInterceptor(demoInterceptor)
                .addPathPatterns("/**")//设置拦截器拦截的请求路径（ /** 表示拦截所有请求）
                .excludePathPatterns("/login");//设置不拦截的请求路径
```

| 拦截路径  | 含义                 | 举例                                                |
| --------- | -------------------- | --------------------------------------------------- |
| /*        | 一级路径             | 能匹配/depts，/emps，/login，不能匹配 /depts/1      |
| /**       | 任意级路径           | 能匹配/depts，/depts/1，/depts/1/2                  |
| /depts/*  | /depts 下的一级路径   | 能匹配/depts/1，不能匹配/depts/1/2，/depts          |
| /depts/** | /depts 下的任意级路径 | 能匹配/depts，/depts/1，/depts/1/2，不能匹配/emps/1 |

执行关系如下:

![ebd86330-3872-48fd-8b4a-37c7ac04cb3c](./day5.assets/ebd86330-3872-48fd-8b4a-37c7ac04cb3c.png)

- 过滤器放行后进入到spring环境中
- Tomcat不能识别Controller,但能识别Servlet程序,因此Spring的Web环境中提供了一个核心的Servlet(DispatcherServlet前端控制器),所有请求都会经由DispatcherServlet转给Controller

过滤器与拦截器的区别:

- **接口规范不同：过滤器需要实现Filter接口，而拦截器需要实现HandlerInterceptor接口。**
- **拦截范围不同：过滤器Filter会拦截所有的资源，而Interceptor只会拦截Spring环境中的资源。**
