## SpringBoot 高级操作

## 1. SpringBoot 配置文件的读取

***<u>==SpringBoot 读取配置文件的方式:==</u>***

1. 通过 ==@Value==
2. 通过 @ConfigurationProperties(prefix = "配置文件前缀")

### 1.1 @Value 读取配置文件



### 1.2 @ConfigurationProperties 读取配置文件



#### 1.2.1 从默认配置文件读取配置项

如果不指定, ==@ConfigurationProperties== 会总默认的配置文件 *==application.properties 或者 application.yml==* 中读取配置

==<u>***使用方式:***</u>==

1. 在配置文件中添加配置项.
2. 编写配置项读取类, 使用注解 ==***@ConfigurationProperties***==, 并指定前缀. 同时使用 ==***@Component***== 将配置项读取类交给框架管理.
3. 在配置项读取类中, 添加属性, 属性名要和配置项的名称一样, 然后提供 ==***get/set***== 方法.
4. 在要是用的地方注入配置项读取类.

**<u>注意: 只能在同样交给框架管理的类中使用配置项读取类. 比如 Controller 中, 在 test 类中获取不到.</u>**

**==*application.properties/application.yml*==**

```properties
security.basic.page.singin = /staitc/singin1.html
```

***==SecurityConfigurationProperties 读取配置参数==***

```java
@Component
@ConfigurationProperties(prefix = "security.basic.page")	// 配置项的前缀
public class SecurityConfigurationProperties {
		// 如果配置了此配置项, 则使用配置项的值, 如果没有配置, 则使用默认的值
    private String singin = "/static/login.html";
  
    public String getSingin() {
        return singin;
    }

    /**
     * @param singin set singin
     */
    public void setSingin(String singin) {
        this.singin = singin;
    }
}
```

***==使用==***

```java
@Slf4j
@RestController
public class SecurityController {

    @Autowired
    private SecurityBasicProperties securityBasicProperties;

    @RequestMapping("/required")
    public SimpleResponse required(){
        String singin = securityBasicProperties.getSingin();
        log.info(singin);
        return new SimpleResponse(singin);
    }
}
```



#### 1.2.2 从指定配置文件读取配置项

在配置项读取类上添加注解 ***==@PropertySource==***, 并指定要读取的配置文件.

```java
@Component
@PropertySource("classpath:env.properties")	// 指定从 classpath 下的 env.properties 中读取配置项
@ConfigurationProperties(prefix = "security.basic.page")	// 配置项的前缀
public class SecurityConfigurationProperties {
		// 如果配置了此配置项, 则使用配置项的值, 如果没有配置, 则使用默认的值
    private String singin;

    public String getSingin() {
        return singin;
    }

    public void setSingin(String singin) {
        this.singin = singin;
    }
}
```

#### 

#### 1.2.3 配置项读取类里边的属性也是个类.

当有很多种类型的配置参数时, 比如有图片验证码的配置, 短信验证码的配置, 登录项 的配置等, 如果都放在一个配置项读取类里边会过于臃肿, 所以可以对应多种配置分别封装多个配置类, 然后在一个总的配置类中声明这几种配置项读取类的参数.

图片验证码配置类

```java
@Data
public class ImageCodeProperties {
    // 图片宽度
    private int width = 67;
    // 图片高度
    private int height = 23;
    // 验证码长度
    private int lenght = 4;
    // 验证码有效期 秒
    private int expireIn = 60;
    // 需要进行图片验证的 URL
    private String url;
}
```

短信验证码配置类

```java
@Data
public class PhoneCodeProperties {
    private int length = 5;
}
```

验证码参数配置类

```java
@Data
public class ValidateCodeProperties {
    // 图片验证码
    private ImageCodeProperties image = new ImageCodeProperties();
    // 手机验证码
    private PhoneCodeProperties phone = new PhoneCodeProperties();
}
```

登录项配置类

```java
@Data
public class AuthenticationBasicProperites {
    private String singin = "/static/login.html";

    private LoginType loginType = LoginType.JSON;
}
```

总的参数配置类

```java
@Data
@Component
@PropertySource("classpath:env.properties")
@ConfigurationProperties(prefix = "security.basic.prp")
public class SecurityBasicProperties {

    // 登录认证配置参数
    private AuthenticationBasicProperites authentication = new AuthenticationBasicProperites();

    // 验证码配置参数
    private ValidateCodeProperties code = new ValidateCodeProperties();
}
```

配置文件

```properties
# 登录认证配置
security.basic.prp.authentication.singin = /static/singin1.html
security.basic.prp.authentication.login-type= json

# 验证码配置
security.basic.prp.code.image.width = 6
security.basic.prp.code.image.length = 100
security.basic.prp.code.image.url = /user,/user/*
security.basic.prp.code.phone.length = 5

# 说明:
# security.basic.prp 对应 ConfigurationProperties 配置的 prefix
#	authentication, code 分别对应 ConfigurationProperties 的属性 : AuthenticationBasicProperites 和 ValidateCodeProperties
#	image, phone 分别对应 ValidateCodeProperties 的属性 : ImageCodeProperties 和 PhoneCodeProperties
# 验证码配置项中最后的字段就对应实体内的属性名
```

使用

```java
@Autowired
private SecurityBasicProperties securityBasicProperties;
```



## 2. SpringBoot 页面等资源的存放位置

#### 2.1 默认位置

==***<u>默认情况下, SpringBoot 提供了 5 个位置来访静态资源:</u>***==

1. ==*classpath:/META-INF/resources/*==
2. ==*classpath:/resources/*==
3. ==*classpath:/static/*==
4. ==*classpath:/public/*==
5. ==*/*==

以上 5 个位置, 是有**优先级**的, 加入同一个文件分别存在于这 4 个位置, 那么优先级也是按照上边的顺序来的.



比较特殊的是 **==*classpath:/static/*==**

系统默认创建了 ==*classpath:/static/*==, 正常情况下, 我们可以直接把资源放在这个路径下, 不需要额外的创建, 但是访问 ==*classpath:/static/*== 路径下的资源时, 不需要加 *==/static==*, 这是因为在 *==WebMvcAutoConfiguration#addResourceHandlers==* 方法中, *==this.resourceProperties.getStaticLocations()==* 会返回 *==ResourceProperties#staticLocations==*, 这个属性封装了这 5 中路径,自动添加了 *==static==*. 

所以当我们在 ==*classpath:/static/*== 路径下放入一个静态资源的时候, 例如是 *==login.html==*, 访问路径应该是 ==localhost:8080/login.html==, 不用添加 *==/static==*, 系统会自动去==*classpath:/static/*== 路径下查找 ==login.html==.

#### 2.2 自定义配置路径

```properties
# 指定资源的位置
spring.resources.static-locations=classpath:/
# 定义请求 URL 规则, 此时如果是放在 static 下的资源, 访问的时候需要使用全路径, 以上边的为例, 应该是 : localhost:8080/static/login.html
spring.mvc.static-path-pattern=/**
```



## 3.基于 SpringBoot 的服务拦截

**SpringBoot 服务拦截有三种方式**

1. 拦截器
2. 过滤器
3. 切片

### 3.1 基于 SpringBoot 的过滤器
#### 3.1.1 Filter 编写
1. 实现 Filter 接口, 并重写方法
```java
@Slf4j
public class CustomFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("filter --> init() : init");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        log.info("filter --> doFilter() : starting");
        filterChain.doFilter(servletRequest, servletResponse);
        log.info("filter --> doFilter() : end");
    }

    @Override
    public void destroy() {
        log.info("filter --> destroy(): destroy");
    }
}
```
#### 3.1.2 引入 Filter
==方式一: 在 Filter 上使用 @Component 注解==
这种方式比较简单, 直接添加==@Component== 注解, 将 Filter 交给框架管理就行了, 这时Filter 就会起作用.
==**但是这种方式会过滤所有的 URL**==, 无法通过代码过滤指定的 URL. 
如果想通过代码过滤指定的 URL 可以使用方式二.

==方式二: 在配置类中, 返回 FilterRegistrationBean==
这种方式可以通过在==@Configuration== 配置类中, 使用==**@Bean**== 返回==**FilterRegistrationBean**== 来将过滤器添加到项目的过滤器链中.
==**并且可以通过代码指定要过滤的 URL**.==
```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean customFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new CustomFilter());
        List<String> urls = new ArrayList<>();
        urls.add("/user/*");
        filterRegistrationBean.setUrlPatterns(urls);
        return filterRegistrationBean;
    }
}
```

### 3.2 基于 SpringBoot 的拦截器
#### 3.2.1 拦截器作用
1. 在方法执行前, 方法执行后渲染视图前, 渲染视图后返回视图前三个时间点执行方法.
2. 可以获取到请求(不是真正要调用的 Controller 方法的参数), 响应, 以及真正要调用的 Controller 的方法对象.
3. 三个方法参数列表中的 ==*Object handler*==都是真正要调用的 Controller 方法的声明
4. ==**会拦截所有的方法, 不仅仅是自己编写的, 其他的也会拦截.**==

==**需要注意**==:
1. 拦截器获取不到要调用的 Controller 的方法的参数.
##### 3.2.2 拦截器编写
1. 实现 ==HandlerInterceptor==接口, 并重写三个方法
```java
@Slf4j
@Component
public class CustomInterceprot implements HandlerInterceptor {
    /**
     * 请求方法执行之前执行
     * @param httpServletRequest
     * @param httpServletResponse
     * @param handler 真正要执行的请求方法的声明
     * @return fasle 则会不调用真正要调用的方法. true 会调用真正要执行的方法.
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object handler) throws Exception {
        log.info("interceptor --> preHandle() : 请求执行之前运行");
        HandlerMethod handle = (HandlerMethod)handler;
        String className = handle.getBean().getClass().getName();
        String methodName = handle.getMethod().getName();
        log.info("interceptor --> preHandle() : handler className: [{}], methdoName : [{}]”, className, methodName);
        return true;
    }

    /**
     * 真正要执行的方法执行完毕, 视图渲染之前执行.如果真正要执行的方法抛出了异常, 那么这个方法就不会被执行
     * @param httpServletRequest
     * @param httpServletResponse
     * @param o
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        log.info("interceptor -->  postHandle(): 方法执行完毕之后, 渲染视图之前执行");

    }

    /**
     * 视图渲染后, 返回执行执行. 无论方法是否抛出异常, 此方法都会执行
     * @param httpServletRequest
     * @param httpServletResponse
     * @param o
     * @param e
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        log.info(“interceptor —> afterCompletion() : 视图渲染后调用”);
    }
}
```
#### 3.2.3 引入 Interceptor
1. 编写 Interceptor , 并使用 ==@Component== 将拦截器交给框架管理.
2. 编写配置类, 并标注 ==@Configuration==, 同时继承 ==**WebMvcConfigurerAdapter**==, 重写 ==**addInterceptors**== 方法.
3. 在配置类中 ==@Autowired== 注入拦截器.
4. 在 ==addInterceptors==方法中将拦截器添加到拦截器链中去
```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

    @Autowired
    private CustomInterceprot customInterceprotl;

    /**
     * 配置 Interceptor 拦截器, 配置类需要继承 WebMvcConfigurerAdapter 类, 并重写此方法
     * 然后注入编写的拦截器, 并添加到拦截器链中
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(customInterceprotl);
    }

    /**
     * 配置Filter 过滤器, 不用继承 WebMvcConfigurerAdapter 类.
     * @return
     */
    @Bean
    public FilterRegistrationBean customFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new CustomFilter());
        List<String> urls = new ArrayList<>();
        urls.add(“/user/*”);
        filterRegistrationBean.setUrlPatterns(urls);
        return filterRegistrationBean;
    }
}
```

### 3.3 切片
拦截器获取不到真正要调用的 Controller 的方法的参数, 如果想获取到 Controller 方法的参数, 可以使用**==切片==**的方式进行拦截.
其实就是==**AOP**==
==**需要注意两个点**==:
1. **==切入==**点 : 在哪些方法上起作用,  什么时候起作用
2. ==**增强**==: 起作用时所要执行的业务逻辑
满足以上两点后, 满足条件的方法就会执行增强.

#### 3.3.1 切片编写
1. 编写切片类, 不需要实现或者继承任何类, 使用注解 : @Aspect, @Component
2. 在切片类中编写增强
```java
@Slf4j
@Aspect
@Component
public class CustomAspect {
//
//    @Before(“execution(* *.*(..))”)
//    public Object beforeHandle(){
//
//    }
//
//    @After("execution(* *.*(..))")
//    public Object afterHandle(){
//
//    }
//
//    @AfterThrowing("execution(* *.*(..))")
//    public Object AfterThrowing(){
//
//    }
//
//    @AfterReturning("execution(* *.*(..))")
//    public Object AfterReturning(){
//
//    }


    @Around("execution(* com.example.securitydemo.controller.restful.*.*(..))")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        log.info("Aspect --> @Around : round start");
        Object[] args = pjp.getArgs();
        log.info("Aspect --> @Around : 参数列表  --> @Around : [{}]", args);
        Object proceed = pjp.proceed();
        log.info(“Aspect —> @Around : Controller 返回值 : [{}]”, proceed);
        log.info(“Aspect —> @Around : round end”);
        return proceed;
    }
}
```



## 4. SpringBoot 配置

### 4.1 以 @Bean 的形式管理 bean

需要用到的注解:

- @Configuration
- @Bean
- @Conditionalxxxx (条件化操作)



要交给 Spring 管理的 bean

```java
// 接口
interface ValidateCodeGenerator {
  public void showMessage();
}

// 实现
class ImageCodeGenerator implements ValidateCodeGenerator{
   @Override
  public void showMessage(){
    sout("123");
  }
}
```



配置

```java
@Configuration
public class ValidateCodeGeneratorConfig {

  	// 当 Spring 容器中不包含名称为 imageCodeGenerator 的 bean 时,
  	// 向 Spring 容器中添加一个名称为 imageCodeGenerator 的 bean, 类型是 ValidateCodeGenerator
    @Bean
    @ConditionalOnMissingBean(name = "imageCodeGenerator")
    public ValidateCodeGenerator imageCodeGenerator(SecurityBasicProperties securityBasicProperties){
        ImageCodeGenerator imageCodeGenerator = new ImageCodeGenerator();
        imageCodeGenerator.setSecurityBasicProperties(securityBasicProperties);
        return imageCodeGenerator;
    }
}
```



此时直接注入运行的话, 结果为 123.



如果此时再编写一个 ValidateCodeGenerator 的实现, 并且使用 @Component("imageCodeGenerator"), 那么上边的配置就不会向 Spring 容器中添加 bean

```java
@Component("imageCodeGenerator")
public class ImageCodeGenerator2 implements ValidateCodeGenerator {

    @Override
    public void showMessage(){
        System.out.println("456");
        return null;
    }
}
```

那么此时, 执行的就是 ImageCodeGenerator2 的 showMessage 方法, 结果是 456.