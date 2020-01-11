---

layout:     post   				    # 使用的布局（不需要改）
title:      SpringMVC的helloworld				# 标题 
subtitle:   一切都是对象 #副标题
date:       2019-12-26 				# 时间
author:     liberty 						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    -Spring
    -Springmvc
	-Java
---

## SpringMVC的HelloWorld

参考链接：[IDEA建立Spring MVC Hello World 详细入门教程](https://www.cnblogs.com/wormday/p/8435617.html)

### 创建项目
勾选Spring-SpringMVC,JAVAEE-WebApplication，创建项目。在项目名称右键，Add Framework Support。添加maven。

pom内容如下：[pom.xml](2019-12-26-pom.xml.md)

项目结构如下

```
.
├── mvc-hello.iml
├── pom.xml
├── src
│   └── com
│       └── sitech
│           └── HiController.java
└── web
    ├── WEB-INF
    │   ├── applicationContext.xml
    │   ├── dispatcher-servlet.xml
    │   ├── jsp
    │   │   └── say.jsp
    │   └── web.xml
    └── index.jsp
```

#### HiController.java

```java
package com.sitech;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model; // 这里导入了一个Model类
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/hi")
public class HiController {

    @RequestMapping("/say")
    public String say(Model model) { // 参数中传入Model
        model.addAttribute("name","wormday"); // 指定Model的值
        model.addAttribute("url","http://www.cnblogs.com/wormday/p/8435617.html"); // 指定Model的值
        return "say";
    }
}
```

#### applicationContext.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

#### dispacher-servlet.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

        <context:component-scan base-package="com.sitech"/>
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                <property name="prefix" value="/WEB-INF/jsp/"/>
                <!-- 视图名称后缀  -->
                <property name="suffix" value=".jsp"/>
        </bean>
</beans>

```

#### say.jsp

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

        <context:component-scan base-package="com.sitech"/>
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                <property name="prefix" value="/WEB-INF/jsp/"/>
                <!-- 视图名称后缀  -->
                <property name="suffix" value=".jsp"/>
        </bean>
</beans>
```

### 添加tomcat配置

### 添加artifact

选择“项目名-exploded”，因为是使用maven，所以不用手动添加依赖

### 运行

点击运行，访问路径：http://localhost:8080/hi/say

