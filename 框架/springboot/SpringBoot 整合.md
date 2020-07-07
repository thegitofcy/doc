

# 1. 整合 JSP

### 1.步骤概述

1. 添加依赖.(servlet, jstl, tomcat, jsp 引擎)
2. 创建存放 jsp 页面的目录 `webapp/WEB-INF/jsp`(和 src, resources 同级)
3. 配置解析 jsp 的前缀和后缀
4. 此时就可以通过 controller 进行访问了.

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

在 `resources` 同级目录下创建 `webapp/WEB-INF/jsp` 目录



### 3.配置JSP 前缀后缀

```properties
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```



### 4.编写 Controller 以及创建 home.JSP

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jstl/core_rt"%>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>this is jsp</h1>
    <span>${param}</span><br>	<!-- 传单个属性,需要从指定域中获取, 这样是取不到的 -->
    <span>${requestScope.param}</span><br> <!-- 传单个属性,需要从指定域中获取, requestScope 是指 request 域 -->
    <span>${user.name}</span> <!-- 传递对象可以直接取 -->
</body>
</html>
```



```java
@Controller
public class JSPController {

    @RequestMapping("hello")
    public String hello() {
        return "index";
    }

    @RequestMapping("/modelAndView")
    public ModelAndView modelANdViewTest() {
        ModelAndView modelAndView = new ModelAndView("index");
        modelAndView.addObject("user", new User(1,"老王", "12"));
        modelAndView.addObject("param", "this is modelAndView value");
        return modelAndView;
    }

    @RequestMapping("/model")
    public String modelTest(Model model) {
        model.addAttribute("user", new User(1,"老王", "12"));
        model.addAttribute("param", "this is model value");
        return "index";
    }
}
```



然后就可以访问了.



# 2. 整合 mybatis

### 1. 步骤概述

1. 添加依赖(web, MySQL, mybatis)
2. 

### 2. 添加依赖

```xml
				<!--web核心依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--mysql数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>
```

