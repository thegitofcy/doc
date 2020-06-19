

# SpringBoot 整合 JSP

### 1.添加依赖

```xml
<!-- 添加servlet依赖模块 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>
        <!-- 添加jstl标签库依赖模块 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
        </dependency>
        <!--添加tomcat依赖模块.-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <!-- 使用jsp引擎，springboot内置tomcat没有此依赖 -->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <scope>provided</scope>
        </dependency>

```



### 2.创建目录

在 `resources` 目录下创建 `webapp/WEB-INF/jsp` 目录



### 3. 配置JSP 前缀后缀

```properties
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```



### 4. 编写 Controller 以及创建 home.JSP

```java
@Controller
public class CategoryController {
  
    @RequestMapping("/home")
    public String listCategory(){
        return "home";
    }
}
```



然后就可以访问了.



# 整合