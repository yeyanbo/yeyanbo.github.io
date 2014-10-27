---
layout: post
title: 使用Apache Wink开发RESTful服务
summary: 记录了学习Wink开发RESTful服务的一些心得
tags: [wink RESTful java]
---

这是一个 Java 工程，为了提供 RESTful 服务进行的开发。使用了 Apache Wink 组件。

这是一个示例代码，仅证明可行性及简单性。

##基础篇

### 开发环境

* 开发工具: Eclipse for J2EE
* 应用服务器: Tomcat 6
* 选用基于JAX-RS标准的开源实现: Apache Wink 1.4

### 开发步骤

1. 新建项目,选择 Dynamic Web Project

2. 将如下jar包加入到Web项目中:

	* wink-common-1.4.jar
 	* wink-server-1.4.jar
	* jsr311-api-1.1.1.jar
   	* slf4j-api-1.6.1.jar
   	* slf4j-simple-1.6.1.jar

3. 如果需要支持JSON格式,请加入:

	*  wink-json-provider-1.4.jar
	*  json-20080701.jar
	
	如果不加入这些jar包,请求JSON格式内容时会报错:

	``The resource identified by this request is only capable of generating responses with characteristics not acceptable according to the request "accept" headers ().``

4. 如果需要增加RequestHandler拦截器,请加入:

	* commons-lang-2.3.jar

	如果不加入, 会报错:
	``java.lang.ClassNotFoundException: org.apache.commons.lang.ClassUtils``

5. 代码注解:

````java
	@Path("/SecurityEvent")
	public class SecurityEventService {
````

class上一定要有个@Path, 指名在URL根地址之后的路径, 如果不需要路径,也要如下写:

````java
	@Path("/")
	public class SecurityEventService {
````


如下定义一个简单的资源user, @GET代表HTTP动作, @Produces表示希望的输出格式:


这个输出为XML格式:

````java
	@GET
	@Path("/user")
	@Produces(MediaType.APPLICATION_XML)
	public User getUserInXML() {
		return getUser("unknown");
	}
````

这个输出为JSON格式:

```java
	@GET
	@Path("/user")
	@Produces(MediaType.APPLICATION_JSON)
	public User getUserInJSON() {
    		return getUser("unknown");
	}
```

	注: 他们调用的可是同一个getUser函数哦.

注意了，要想让 User 同时支持 XML 和 JSON 格式，User 类的定义也有讲究：

```java
package com.xxxx.service;

import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement
@XmlAccessorType(XmlAccessType.PROPERTY)
public class User {

	private String name = null;
	private int age = 20;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}
}
```


看到了吗？User 类要用 JAXB 注解进行标记哟，不要忘记了！


再来看看如何处理 List 对象：

```java
package com.xxxxx.testplat.service;

import java.util.List;

import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlElementWrapper;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement
@XmlAccessorType(XmlAccessType.FIELD)
public class MyResult {

	private String status = "PASS";

	@XmlElementWrapper(name = "userlist")
	@XmlElement(name = "user")
	private List<User> users;

	public String getStatus() {
		return status;
	}

	public void setStatus(String status) {
		this.status = status;
	}

	public List<User> getUsers() {
		return users;
	}

	public void setUsers(List<User> users) {
		this.users = users;
	}

}
```

先看看如何获取这个资源

我们假设URL根地址为: ``http://localhost:8080/SecurityEventService``

我们需要先配置下web.xml文件:

```xml
<web-app>
  <display-name>SecurityEventService</display-name>
 
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
  </welcome-file-list>

  <servlet>
    <servlet-name> SecurityEventService</servlet-name > 
    <!--  这个Servlet是Wink提供的  -->
    <servlet-class> 
       org.apache.wink.server.internal.servlet.RestServlet
    </servlet-class> 
    <init-param> 
       <param-name> applicationConfigLocation</param-name > 
       <param-value> /WEB-INF/application</param-value > 
    </init-param> 
  </servlet> 

  <servlet-mapping> 
    <servlet-name> SecurityEventService</servlet-name > 
    <!-- 这个是Servlet对应的path  -->
    <url-pattern> /rest/*</url-pattern>                        
  </servlet-mapping> 
</web-app > 
```

附属的还有的文件: /WEB-INF/application
指名拥有资源的类: 内容目前只有一行:
``com.wondersgroup.core.audit.SecurityEventService``

OK, 可以运行了.

在浏览器中输入地址:
``http://localhost:8080/SecurityEventService/rest/user``

返回如下:

```xml
Content-Length →132
Content-Type →application/xml
Date →Mon, 14 Jul 2014 11:40:01 GMT
Server →Apache-Coyote/1.1

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<user pin="208">
    <password>23667</password>
    <username>unknown</username>
</user>
```

那么如何获得JSON格式的内容呢?

增加Http Request Header参数:

```
Accept: application/json
```

就可以得到JSON格式的内容了:

```json
Content-Type →application/json
Date →Mon, 14 Jul 2014 11:11:01 GMT
Server →Apache-Coyote/1.1
Transfer-Encoding →chunked

{
    "user": {
        "password": "91790",
        "pin": "710",
        "username": "unknown"
    }
}
```


前端页面通过 Jquery AJAX 调用：

```javascript
$.ajax({
     url: "rest/user",
     type: "get",
     dataType: "json",
     success: function(data){
          $("#user").append("User:<br/>");
          $("#user").append("---name:" + data.user.name);
     }
});
```

## 提高篇

### 替换JSON处理引擎

Wink自带的JSON引擎（JSON.org）目前已知存在几个缺陷，使得在RESTful服务开发过程中存在诸多不便。其中，最主要的就是：

1. 不能在函数中直接返回List列表，必须再建一个类进行包装；
2. 数组在数量为0，1，2+时表现形式不一致，使得客户端处理不便；

在[IBM Developerworks](http://www.ibm.com/developerworks/cn/web/wa-aj-jackson/)中介绍了使用Jackson引擎进行替换的方法。

### 增加认证处理拦截器

