# 1. 自定义标签使用场景

当 JSP 内置标签和 jstl 标签库内的标签都满足不了需要的时候, 这时候可以自定义标签.

例如 select 标签, 很多时候在多个页面要使用同样的 select, 会比较麻烦, 这时就可以自定义标签, 然后将 option 值写在字典表内, 通过自定义标签自动获取.



# 2. 自定义标签涉及类

1. `javax.servlet.jsp.tagext.SimpleTagSupport`

   



# 3. 自定义标签实现步骤

1. 编写继承 `TagSupport`的标签处理类. 用于处理标签.(也可以继承 SimpleTagSupport, 重写 doTag 方法.)

2. 重写父类 `setPageContext` 方法, 用于 ==**得到当前 JSP 页面的 pageContext 对象**==.

3. 重写父类 `doStartTag` 方法. 里边是 ==**自定义的标签的java 操作**==.

4. 在 `WEB-INF` 下创建一个 `tld`类型的文件. 此文件用于注册自定义标签.(如果文件没有在 web-inf 下, 则需要在 web.xml 中通过 taglib 进行配置)

5. 在要使用自定义标签的 jsp 页面引导入自定义的标签库

   `<%@ taglib uri="/WEB-INF/lems-tags/lems-reference.tld" prefix="lems"%>`





# 4.自定义标签原理

1. 服务器打开时, 会加载 web-inf 下的资源文件, 包括 web.xml 和 tld 文件, 把他们加载到内存.
2. 访问 jsp 页面
3. 服务器读取到页面的内容, 当读取到 `<%@ taglib uri="/WEB-INF/lems-tags/lems-reference.tld" prefix="lems"%>`时, 就会去内存中找是否存在这个tld 文件, 找不到就报错.
4. 继续往下读取 jsp 文件. 当读到自定义的标签的时候,就会通过 uri 去找 tld 文件, 在 tld 文件中找这个标签, 找到后就得到他的 `tag-class` 的内容, 然后去它对一个的标签处理程序.
5. 实例化标签处理程序, 利用生成的对象调用它里边的方法.





# 5. 代码演示

## 1. Tag 处理类

```java
@Slf4j
@Data   /** 标签属性需要提供 get/set 值 .*/
public class TagProcess extends SimpleTagSupport {

    /** 标签属性. 页面标注属性后, 会自动将属性值映射到此属性上. .*/
    private String id;

    /** 标签处理方法 .*/
    @Override
    public void doTag() throws JspException, IOException {
        log.info("TagProcess run doTage...");
       getJspContext().getOut().write("hello tld");
    }
}
```



## 2. 定义 TID 文件

```xml
<?xml version="1.0" encoding="ISO-8859-1" ?>
<!DOCTYPE taglib
        PUBLIC "-//Sun Microsystems, Inc.//DTD JSP Tag Library 1.2//EN"
        "http://java.sun.com/dtd/web-jsptaglibrary_1_2.dtd">
<taglib>
    <tlib-version>1.0</tlib-version>
    <jsp-version>1.2</jsp-version>
    <short-name>ttag</short-name>   <!-- 标签库的简称 -->
    <uri>http://vmaxtam.com/mytag</uri> <!-- 引入此标签库使用的 URI -->

    <!-- 定义标签库的标签, 多个标签就是多个  tag 标签 -->
    <tag>
        <name>testTag</name> <!-- 标签名称 -->
        <tag-class>com.cy.core.util.tag.TagProcess</tag-class> <!-- 处理此标签的 tag 处理类 -->
        <body-content>empty</body-content>
        <attribute> <!-- 标签的属性, 多个属性就是多个 attribute -->
            <name>id</name> <!-- 属性名称 -->
            <required>false</required>  <!-- 此属性是否必须 -->
            <rtexprvalue>true</rtexprvalue> <!-- 是否可以使用表达式 -->
        </attribute>
    </tag>
</taglib>
```

- ==**注意**==: 如果定义在 web-inf 下, 则会自动识别. 如果没有定义在 web-inf 下, 则需要在 web.xml 中进行配置

  ```xml
  <jsp-config>
      <taglib>
          <!-- tld描述文件中的uri -->
          <taglib-uri></taglib-uri>
          <!-- 存放路径 -->
          <taglib-location></taglib-location>
      </taglib>
    </jsp-config>
  ```

  

## 3. 使用

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib uri="http://vmaxtam.com/mytag" prefix="ttag"%>`
<!DOCTYPE html>
<html lang="en">
<head>
</head>
<body>
    <ttag:testTag id="testId"/>
</body>
</html>
```

