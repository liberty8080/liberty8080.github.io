---
layout:     post   				    # 使用的布局（不需要改）
title:      Spring 之 IOC,AOP概念整理				# 标题 
subtitle:   Hello World, Hello Blog #副标题
date:       2019-07-26 				# 时间
author:     liberty						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - Java
    - Spring
---


### Spring 之IOC，AOP整理

#### 配置文件方式注入对象

在applicationContext.xml 是Spring的核心配置文件，通过name "c" 即可获取Catogery对象,该对象获取的时候,即被注入字符串”Catogery1 “到name属性中

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
   http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/aop
   http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
   http://www.springframework.org/schema/tx
   http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
   http://www.springframework.org/schema/context     
   http://www.springframework.org/schema/context/spring-context-3.0.xsd">
  
   <bean name="c" class="com.how2java.pojo.Category">
        <property name="name" value="category 1" />
    </bean>
  
</beans>
```

在对象中注入另一个对象,使用ref关键字

```xml
    <bean name="p" class="com.how2java.pojo.Product">
        <property name="name" value="product1" />
        <property name="category" ref="c" />
    </bean>
```

#### 使用注解方式注入对象

##### 1. 对属性的注解  @AutoWired

在applicationContext.xml 中增加

\<context:annotation-config/>

表示用注解的方式进行配置

##### 2. 对Bean本身的注解 @Component

在applicationContext.xml中增加：

<context:component-scan base-package="com.how2java.pojo"/>

告诉Spring ，bean放在这个包下

```xml
package com.how2java.pojo;
 
import javax.annotation.Resource;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
 
@Component("p")
public class Product {
 
    private int id;
    private String name="product 1";
     
    @Autowired
    private Category category;
 
    public int getId() {
        return id;
    }
 
    public void setId(int id) {
        this.id = id;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public Category getCategory() {
        return category;
    }
     
    public void setCategory(Category category) {
        this.category = category;
    }
}
```

