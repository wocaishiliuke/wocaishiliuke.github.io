---
title: Java Web中的版本
date: 2018-10-20 00:00:00
categories:
    - Servlet
tags:
    - Servlet
    - Web
    - Jsp
---

本文由WEB项目部署描述符web.xml的版本问题引出，故对Java中的Web技术栈版本稍作整理。包括Java SE、Java EE、Servlet等

<!-- more -->

##### 目录
+ I.web.xml版本问题
+ II.Java中的版本
+ III.Server版本

---

# I.web.xml版本

实际是该Web项目使用的Servlet版本问题

## 1.问题描述

> 使用IDEA的maven-archetype-webapp骨架或直接使用命令创建Web项目（其实都是maven-archetype-webapp:1.0插件），生成的WEB项目描述符web.xml为2.3版本

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

> 该web.xml 2.3版本，对应Servlet 2.3，过于老旧。如[默认忽略EL表达式](https://docs.oracle.com/cd/E19316-01/819-3669/6n5sg7b0v/index.html)等（不是不支持jsp中的EL，而是忽略，即原样输出。2.4版本及以上才默认不忽略jsp中的EL），所以有时需要更改

```
If isELIgnored is true, EL expressions are ignored when they appear in static text or tag attributes. If it is false, EL expressions are evaluated by the container only if the attribute has rtexprvalue set to true or the expression is a deferred expression.

Each JSP page has a default mode for EL expression evaluation.The default value of isELIgnored varies depending on the version of the web application deployment descriptor. The default mode for JSP pages delivered with a Servlet 2.4 descriptor is to evaluate EL expressions; this automatically provides the default that most applications want. The default mode for JSP pages delivered using a descriptor from Servlet 2.3 or before is to ignore EL expressions; this provides backward compatibility
```

## 2.解决方式

#### 2.1 修改单个页面

在每个页面中使用isELIgnored="false"，不忽略EL表达式

```
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8" isELIgnored="false"%>
```

#### 2.2 修改单个项目

以IDEA为例，修改web.xml的版本2.4及以上（默认识别EL），则jsp页面不需要再指定

###### 方式1：直接替换

直接替换web.xml的头信息

> 各版本web.xml的头声明，参见IDEA中settings中File and Code Templates中Web中的模板，如：

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
</web-app>
```

###### 方式2：借助IDE修改

- 在Project Structure的Facets(or Modules)中，删除对应项目或模块的Web Deployment Descriptor，apply
- 然后添加新的web.xml，选择合适的版本，如3.1

#### 2.3 修改maven-archetype-webapp中的模板

直接修改maven-archetype-webapp:1.0插件中的web.xml模板，只要本地仓库中的该插件不被替换，以后使用IDE或命令创建的maven WEB项目，都会使用修改后的web.xml模板

- step1：本地仓库的插件jar

```
/home/top/maven_repository/org/apache/maven/archetypes/maven-archetype-webapp/1.0
```

- step2：Open With Archive Manager，在archetype-resources/src/main/webapp/WEB-INF下找到web.xml
- step3：修改该模板即可

# II.Java版本

主要有Java SE版本、JDK版本、Java EE版本

## 1.Java SE和JDK

> 关于Java SE和JDK的命名与版本[参考官网](https://www.oracle.com/technetwork/java/javase/namechange-140185.html)。从J2SE 1.5，使用5/6/7...作为发布版本，但仍然保留了1.5.0（1.5）...作为开发版本。所以J2SE 5.0 = J2SE 5 = J2SE 1.5.0

- Java SE是Java的一个平台版本，包含Java的核心API：Collection、I/O、JDBC、i18n等
- 而JDK是开发工具包。JDK和Java SE的版本发布相同
- JDK，全称Java SE Development Kit，区别于Oracle的Java EE Development Kit
- JDK包含了Java SE的JVM和API，额外的还有其他编译、打包等开发工具
- Java EE SDK比JDK更多jar，如Servlet、JSP、JPA、JSTL等，支持企业级开发

> 所以企业级Web开发两种方式：1.基于JDK+Spring等开源框架，2.基于Java EE SDK，目前第一种方式流行，所以大部分使用JDK开发，根据需要额外引入Servlet、JSP等Java EE的相关jar）

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Servlet/note01_01.png)

## 2.Java EE

> - Java EE初期只包含Servlet和JSP，随后JSTL、JSF和EL等才加入
> - 前名J2EE（Java 2 Platform Enterprise Edition），后名Java EE（Java Platform Enterprise Edition），2018.3更名Jakarta EE
> - 1998年发表JDK1.2时，Java开始分为3个平台版本J2SE、J2EE、J2ME
> - 2006年发布的SE平台改名为Java SE 6，EE平台版本也更名为Java EE 5

> James Gosling在1994/1995致力于Web Server的工作，成为了后来servlet的基础。1996年Pavani Diwanji牵头一个更大的项目，Sun公司的Java Web server产品诞生于该项目（最早支持Servlet标准的就是JavaSoft的Java Web Server。此后，一些其它基于Java的Web服务器开始支持Servlet，Tomcat、Jetty等）。1999年1月，以James Davidson为首的Servlet项目组发布了Servlet2.1，同年12月发布了Servlet2.2。而以Larry Cable和Eduardo Pelegri-Llopart为首的JSP项目组，在1999年6月发布了JSP1.0，同年12月发布了JSP1.1

|版本|日期| | | | | |
|:---|:---|:-|:-|:-|:-|:-|
|Servlet 1.1|Jan 1999| | | | | |
|JSP 1.0|Jun 1999| | | | | |
|J2EE 1.2 |1999.12.12|Servlet 2.2|JSP 1.1| | | |
|J2EE 1.3 |2001.9.24|Servlet 2.3|JSP 1.2| | | |
|J2EE 1.4 |2003.11.11|Servlet 2.4|JSP 2.0| | | |
|Java EE 5 |2006.5.11|Servlet 2.5|JSP 2.1|JSTL 1.2|JSF 1.2| |
|Java EE 6 |2009.12.10|Servlet 3.0|JSP 2.2|JSTL 1.2|JSF 2.0|EL 2.2|
|Java EE 7 |2013.6.12|Servlet 3.1|JSP 2.3|JSTL 1.2|JSF 2.2|EL 3.0|
|Java EE 8 |2017.8.31|Servlet 4.0|JSP 2.3|JSTL 1.2|JSF 2.3|EL 3.0|

> - Java EE开始于Servlet，后来加入JSP。即早期的J2EE只包含JSP和Servlet，并命名为J2EE
> - 随后EL、JSTL的出现，是为了简化JSP代码，但它们只是独立的包，此阶段开发时需要单独加入这两个依赖到IDE和Tomcat
> - 在Java EE 5及以后，才将JSTL和JSF引入，但此时对应的Tomcat并没有引入JSTL和JSF，所以使用Tomcat的Java EE项目需要单独引入这两个依赖，还要注意版本对应（参考下文Tomcat版本）。如Tomcat的示例，在webapps/examples/WEB-INF/lib引入的，可参考[Apache Taglibs](http://tomcat.apache.org/taglibs.html)（Apache Standard Taglib是JSTL的实现）
> - 在Java EE 6才引入EL。不像JSTL和JSF，对应的Tomcat也加入了EL依赖

|JEE Version |TOMCAT Version|
|:-----------|:-------------|
|JEE3 support Servlet 2.3 && JSP 1.2|Tomcat4 support the same as JEE3(Servlet 2.3 && JSP 1.2)|
|JEE4 support Servlet 2.4 && JSP 2.0|Tomcat5 support the same as JEE4(Servlet 2.4 && JSP 2.0)|
|JEE5 support Servlet 2.5, JSP 2.1, JSTL 1.2, JSF 1.2|Tomcat6 support Servlet 2.5, JSP 2.1, EL 2.1. No JSTL.No JSF|
|JEE6 support Servlet 3.0, JSP 2.2, EL 2.2, JSTL 1.2, JSF 2.0|Tomcat7 support Servlet 3.0, JSP 2.2, EL 2.2 as well|

> - EE平台版本一般比同版本的SE版本晚。它基于SE，另由一系列技术标准组成。以Java EE 7为例，包含的specifications如下

|标准类别|具体项|
|:-------|:-----|
|Web app|Servlet、JavaServer Pages、Expression Language、JSTL、JavaServer Faces、WebSocket|
|Enterprise app|Dependency injection、Bean Validation、Enterprise JavaBeans、JPA、JMS、JTA、JavaMail、JCA
Common Annotations|
|Web services|JAX-RS、JAX-WS、Web Services Metadata、Java API for XML Messaging、JAXR|
|APM|J2EE Management|
|Related|JAXB、JAXP、JDBC、JMX、JavaBeans Activation Framework、Streaming API for XML|

> - Web profile（Java EE Web Profile SDK）是Java EE（Java EE Platform SDK）的子集，只包含了常用的Servlet、JSP、JSF、EJB、JPA和JTA等规范，不包括JMS、JNDI等不常用的API，更轻量

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Servlet/note01_02.jpg)

## 3.JSP

JavaServer Pages（JSP）是一项基于HTML、XML和其他文件类型，动态生成web页面的技术，由Sun Microsystems在1999发布。JSP和PHP、ASP相似，但它使用的是Java语言。也可以把JSP看做扩展版的Servlet，因为它提供了EL、JSTL等更多功能

> EL和JSTL的出现，是为了简化和方便管理JSP代码。不使用scriplets（<% Java %>）就可以编写JSP

- JSP中包含了HTML tag和JSP tag
- JSP 1.1引入了tag libraries标签库，来扩展standard JSP syntax，其中一个standard tag library，就是JSTL（随后一些MVC框架，如Struct、SpringMVC也引入了相应的标签）
- JSP 1.2提升了对tag libraries的支持，JSP 1.2发布后，JSTL开始形成，JSTL 1.0单独发布（所以JSTL 1.0需要在支持JSP 1.2的JSP Container中使用）。EL则开始于JSTL 1.0。
- JSP 2.0（J2EE 4）开始引入EL。JSP 2.0发布后，JSTL 1.1发布（所以JSTL 1.1需要在支持JSP 2.0的JSP Containe中使用r）。有了JSTL和EL后，编写JSP可以不需要熟悉Java语言，即不使用scriplets，而是使用自定标签
- JSP 2.1（J2EE 5）






#### 参考

- [jsp tutorial](https://www.javatpoint.com/jsp-tutorial)

## 3.EL

> EL原名SPEL（Simplest Possible Expression Language）

- EL起始于JSTL 1.0（而非JSP）。在加入J2EE 5之前，JSTL是独立发展的（没有加入J2EE）
- 2001年发布的JSP 1.2（J2EE 3）期间，市场上还没有EL
- 2002.6.21发布了有EL支持的JSTL1.0
- 随后发布的JSP2.0（J2EE 4）增加了EL支持，JSF也增加了自己的EL版本，所以此时的EL在JSP和JSF中是不同的
- JSP 2.1（J2EE 5）开始支持unified EL（JSP 2.0和JSF 1.0的统一EL版本)。有了unified EL，JSTL 1.2和JSF 1.2都加入了J2EE 5

[参考](https://www.javasprint.com/java_training_tutorial_blog/java_jee_jsp_servlet_story_jsf_jstl_el_history.php)

## 4.JSTL

> [JSTL](https://docs.oracle.com/javaee/5/tutorial/doc/bnake.html)提供了一组tags来简化JSP的开发。

|Area|Prefix|Subfunction|URL|
|:---|:-----|:----------|:--|
|Core|c|Variable support、URL management、Flow control、Miscellaneous|http://java.sun.com/jsp/jstl/core|
|XML|x|Core、Flow control、Transformation|http://java.sun.com/jsp/jstl/xml|
|I18N|fmt|Locale、Message formatting、Number and date formatting|http://java.sun.com/jsp/jstl/fmt|
|Database|sql|SQL|http://java.sun.com/jsp/jstl/sql|
|Functions|fn|Collection length、String manipulation|http://java.sun.com/jsp/jstl/functions|


#### 各版本JSTL的使用

###### JSTL 1.1
```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.1.2</version>
</dependency>
<dependency>
  <groupId>taglibs</groupId>
  <artifactId>standard</artifactId>
  <version>1.1.2</version>
</dependency>
```

###### JSTL 1.2

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

#### 1.5 JSF

JSF(JavaServer Faces)是基于服务器端组件的用户界面框架，面向组件和事件驱动模型，用于开发WEB应用

> 在JavaEE 5开始，JSF 1.2才被引入（同时引入的还有JSTL 1.2，EL在JavaEE 6才被引入）。但JSF和JSTL都没有被随后的Tomcat等barebones Server引入。使用这些服务器时，如果需要，必须单独引入JSF和JSTL的包

JSF的历史版本参考[Wikipedia](https://en.wikipedia.org/wiki/JavaServer_Faces)

|JSF Version|Released in|
|JSF 1.0 |2004.3.11|
|JSF 1.1 |2004.5.27|
|JSF 1.2 |2006.5.11 with JavaEE 5|
|JSF 2.0 |2009.12.10 with JEE 6.It uses Facelets.|
|JSF 2.1 |2010.11.22|
|JSF 2.2 |2013.5.21|
|JSF 2.3 |2017.3.28|


# III.Server版本

## 1.Tomcat

> - Tomcat是Java Servlet和JavaServer Pages技术的实现
> - 不同版本的Servlet和JSP可使用的tomcat版本也不同，对应关系[参考官网](http://tomcat.apache.org/whichversion.html)如下

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Servlet/note01_03.png)

> 最后一栏是Tomcat对JDK版本的要求

## 2.[Jetty](http://www.eclipse.org/jetty/about.html)

|Version|Year|Home|JVM|Protocols|Servlet|JSP|Status|
|:------|:---|:---|:--|:--------|:------|:--|:-----|
|9.3|2015|Eclipse|1.8|HTTP/1.1 (RFC 7230), HTTP/2 (RFC 7540), WebSocket (RFC 6455, JSR 356), FastCGI|3.1|2.3|Stable|
|9.2|2014|Eclipse|1.7|HTTP/1.1 RFC2616, javax.websocket, SPDY v3|3.1|2.3|Stable|
|8|2009-|Eclipse/Codehaus|1.6|HTTP/1.1 RFC2616, WebSocket RFC 6455, SPDY v3|3.0|2.2|Venerable|
|7|2008-|Eclipse/Codehaus|1.5|HTTP/1.1 RFC2616, WebSocket RFC 6455, SPDY v3|2.5|2.1|Venerable|
|6|2006-2010|Codehaus|1.4-1.5|HTTP/1.1 RFC2616|2.5|2.0|Deprecated|
|5|2003-2009|Sourceforge|1.2-1.5|HTTP/1.1 RFC2616|2.4|2.0|Deprecated|
|4|2001-2006|Sourceforge|1.2, J2ME|HTTP/1.1 RFC2616|2.3|1.2|Ancient|
|3|1999-2002|Sourceforge|1.2|HTTP/1.1 RFC2068|2.2|1.1|Fossilized|
|2|1998-2000|Mortbay|1.1|HTTP/1.0 RFC1945|2.1|1.0|Legendary|
|1|1995-1998|Mortbay|1.0|HTTP/1.0 RFC1945|-|-|Mythical|