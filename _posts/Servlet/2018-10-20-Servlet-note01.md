---
title: web.xml版本
date: 2018-10-20 00:00:00
categories:
    - Servlet
tags:
    - Servlet
    - Web
    - Jsp
---

本文介绍WEB项目部署描述符web.xml的版本问题。

<!-- more -->

##### 目录
+ I.问题描述
+ II.解决方式
+ III.各技术版本

---

# I.问题描述

使用IDEA的maven-archetype-webapp架构或直接使用命令创建Web项目（其实都是maven-archetype-webapp:1.0），生成的WEB项目描述符web.xml为2.3版本

```
$ mvn archetype:generate -DgroupId=com.baicai.test -DartifactId=test-maven-web -Dversion=1.0.0-SNAPSHOT -DarchetypeArtifactId=maven-archetype-webapp
```

> 生成的web.xml模板

```
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
</web-app>
```

> 该web.xml 2.3版本过于老旧。如[默认忽略EL表达式](https://docs.oracle.com/cd/E19316-01/819-3669/6n5sg7b0v/index.html)等（不是不支持jsp中的EL，而是忽略，即原样输出。2.4版本及以上才默认不忽略jsp中的EL），所以有时需要更改

```
If isELIgnored is true, EL expressions are ignored when they appear in static text or tag attributes. If it is false, EL expressions are evaluated by the container only if the attribute has rtexprvalue set to true or the expression is a deferred expression.

Each JSP page has a default mode for EL expression evaluation.The default value of isELIgnored varies depending on the version of the web application deployment descriptor. The default mode for JSP pages delivered with a Servlet 2.4 descriptor is to evaluate EL expressions; this automatically provides the default that most applications want. The default mode for JSP pages delivered using a descriptor from Servlet 2.3 or before is to ignore EL expressions; this provides backward compatibility
```

# II.解决方式

## 1.修改单个页面

> 在每个页面中使用isELIgnored="false"，不忽略EL表达式

```
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8" isELIgnored="false"%>
```

## 2.修改单个项目

> 以IDEA为例，修改web.xml的版本2.4及以上（默认识别EL），则jsp页面不需要再指定

#### 方式1 直接替换

> 直接替换web.xml的头信息

> 各版本web.xml的头声明，参见IDEA中settings中File and Code Templates中Web中的模板，如：

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
</web-app>
```

#### 方式2 借助IDE修改

- 在Project Structure的Facets(or Modules)中，删除对应项目或模块的Web Deployment Descriptor，apply
- 然后添加新的web.xml，选择合适的版本，如3.1

## 3.修改maven-archetype-webapp中的模板

> 直接修改maven-archetype-webapp:1.0插件中的web.xml模板，只要本地仓库中的该插件不被替换，以后使用IDE或命令创建的maven WEB项目，都会使用修改后的web.xml模板

- step1：本地仓库找到该插件jar

```
/home/top/maven_repository/org/apache/maven/archetypes/maven-archetype-webapp/1.0
```

- step2：右键Open With Archive Manager，在archetype-resources/src/main/webapp/WEB-INF下找到web.xml
- step3：修改该模板即可

# III.各技术版本

Java EE 1.2 (December 12, 1999)
Java EE 1.3 (September 24, 2001)
Java EE 1.4 (November 11, 2003)
Java EE 5 (May 11, 2006)
Java EE 6 (December 10, 2009)
Java EE 7 (May 28, 2013)
Java EE 8 (August 31, 2017)


Ver	Rel Date	
Servlet 1.1	Jan 1999	
JSP 1.0	Jun 1999	
JEE2 (J2EE 1.2)	12 Dec 1999	Servlet 2.2 JSP 1.1
JEE3 (J2EE 1.3)	24 Sep 2001	Servlet 2.3 JSP 1.2
JEE4 (J2EE 1.4)	11 Nov 2003	Servlet 2.4 JSP 2.0
JEE5	11 May 2006	Servlet 2.5 JSP 2.1 JSTL 1.2 JSF 1.2
JEE6	10 Dec 2009	Servlet 3.0 JSP 2.2 JSTL 1.2 JS 2.0 EL 2.2
JEE7	12 Jun 2013	Servlet 3.1 JSP 2.3 JSTL 1.2 JSF ;2.2 EL 3.0
