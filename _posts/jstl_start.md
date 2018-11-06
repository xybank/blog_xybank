---
title: jstl标签的使用
date: 2018-11-06 16:54:00 
tags: [java] 
categories: 甘明阳
description: 记录jstl标签的使用
---

jstl全称jsp standard tag library，翻译成中文就是jsp标准标签库。由于jsp文件中有大量的java片段会让文件变得很混乱，于是使用jstl标签替换这些java片段。可以让代码变得更加简洁且提高开发速度。下面详细介绍：

##### `<c:out>`标签

######引用

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
```

######基本使用

作用是用来输入信息的，例如下面输入hello world：

```jsp
<c:out value="hello world"></c:out>
```

######域对象变量

```java
<c:out value="${abc}" default="没找到值"></c:out>
```

如果有四个域对象里面都有相同的变量，那么优先级是：

pageContext > request > session > application

并且如果在在域对象里面找不到对应的变量名则会打印default的值，也可以输出对象，同时也可以用el表达式替换，例如

```jsp
<c:out value="${user1.name}"></c:out>
```

等同于

```jsp
${user1.name}
```

######escapeXml

escapeXml属性表示是否解析跳过html标签，默认值为true

```java
pageContext.setAttribute("abc","你好1<h1>哈哈</h1>");
```

```jsp
<c:out value="${abc}" default="没找到值" escapeXml="false"></c:out>
```

##### `<c:set>`标签

用来设置属性，和`<c:out>`标签差不多的用法

例如：

```jsp
<c:set var="abc" value="行号" scope="request"></c:set>
```

等同于：

```java
request.setAttribute("abc","行号");
```

##### `<c:remove>`标签

用来删除属性的

```jsp
<c:remove var="abc" scope="session" />
```

表示在session作用域里面删除变量abc，如果不写scope则会在所有作用域里面删除abc

##### `<c:catch>`标签

用来抛异常的，例如：

```jsp
 <c:catch var="exception">
	 <%int i = 9/0; %>
 </c:catch>
 <c:out value="${exception.message}"></c:out>
```

##### `<c:if>`标签

单分支判断，例如：

```jsp
<%
    User u = new User();
    u.setName("小明");
    u.setAge(30);
    request.setAttribute("user1", u);
%>
<c:if test="${user1.name=='小明'}">
   测试通过
</c:if>
```

##### `<c:choose>`标签

多分支判断

```jsp
<%
    User u = new User();
    u.setName("小明");
    u.setAge(40);
    request.setAttribute("user1", u);
%>
<c:choose>
     <c:when test="${user1.age < 32}">
         年龄小于32岁
     </c:when>
     <c:when test="${user1.age < 50}">
         年龄小于50岁
     </c:when>
     <c:otherwise>
         年龄大于50岁
     </c:otherwise>
</c:choose>
```

##### `<c:forEach>`标签

######替换for:each

```jsp
<%
    ArrayList<User> list = new ArrayList<User>();
    User u = new User("小明",40);
    User u1 = new User("历史",20);
    User u2 = new User("小红",18);
    User u3 = new User("小李",30);
    list.add(u);
    list.add(u1);
    list.add(u2);
    list.add(u3);
    request.setAttribute("list", list);
%>
<c:forEach items="${list}" var="user">
    <c:out value="${user.name}"></c:out>
    <c:out value="${user.age}"></c:out>
</c:forEach>
```

items表示去迭代的变量，相当于下面的list1，var表示迭代后赋值的变量，相当于下面的user2

```java
 for(User user2: list1){
        System.out.println(user2.getName() + "," + user2.getAge());
    }
```

######替换普通for循环

例如打印1到10

```jsp
<c:forEach var="i" begin="1" end="10">
    <c:out value="${i}"></c:out>
</c:forEach>
```

可以间隔打印，指定step属性，例如下面打印1到10的所有奇数

```jsp
<c:forEach var="i" begin="1" end="10" step="2">
    <c:out value="${i}"></c:out>
</c:forEach>
```

##### `<c:forTokens>`标签

用来拆分字符串的，items表示目标字符串，delims表示根据什么来拆分，var表示拆分出来的字段， 例如：

```jsp
<c:forTokens items="12;56;78;你好" delims=";" var="temp">
    ${temp}
</c:forTokens>
```

在jstl标签里面使用el表达式是非常重要的，在el表达式里面判断某个变量是否为null或者""或者{}都可以用

empty 变量的形式来判断，而且el表达式可以获取a标签传过去的参数，例如${param.id}就是获取上个页面传过来的id

##### `<c:fn>`标签

el表达式也支持函数的使用，不过需要引用jstl函数库，不然会报错，下面是一个拆分手机号码的例子：

```jsp
<!-- 引入jstl函数库 -->
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions"%>
<div class="m_t10 style_dan">手机号：<span>${fn:substring(user.phone, 0,3) }****${fn:substring(user.phone, 7, 11) }</span></div>
```

下面是所有函数表格

| **函数名**            | **函数说明**                                         | **使用举例**                                                 |
| --------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| fn:contains           | 判断字符串是否包含另外一个字符串                     | `<c:if test="${fn:contains(name, searchString)}">`           |
| fn:containsIgnoreCase | 判断字符串是否包含另外一个字符串(大小写无关)         | `<c:if test="${fn:containsIgnoreCase(name, searchString)}">` |
| fn:endsWith           | 判断字符串是否以另外字符串结束                       | `<c:if test="${fn:endsWith(filename, ".txt")}">`             |
| fn:escapeXml          | 把一些字符转成XML表示，例如<字符应该转为&lt;         | ${fn:escapeXml(param:info)}                                  |
| fn:indexOf            | 子字符串在母字符串中出现的位置                       | ${fn:indexOf(name, "-")}                                     |
| fn:join               | 将数组中的数据联合成一个新字符串，并使用指定字符格开 | ${fn:join(array, ";")}                                       |
| fn:length             | 获取字符串的长度，或者数组的大小                     | ${fn:length(shoppingCart.products)}                          |
| fn:replace            | 替换字符串中指定的字符                               | ${fn:replace(text, "-", "&#149;")}                           |
| fn:split              | 把字符串按照指定字符切分                             | ${fn:split(customerNames, ";")}                              |
| fn:startsWith         | 判断字符串是否以某个子串开始                         | `<c:if test="${fn:startsWith(product.id, "100-")}">`         |
| fn:substring          | 获取子串                                             | ${fn:substring(zip, 6, -1)}                                  |
| fn:substringAfter     | 获取从某个字符所在位置开始的子串                     | ${fn:substringAfter(zip, "-")}                               |
| fn:substringBefore    | 获取从开始到某个字符所在位置的子串                   | ${fn:substringBefore(zip, "-")}                              |
| fn:toLowerCase        | 转为小写                                             | ${fn.toLowerCase(product.name)}                              |
| fn:toUpperCase        | 转为大写字符                                         | ${fn.UpperCase(product.name)}                                |
| fn:trim               | 去除字符串前后的空格                                 | ${fn.trim(name)}                                             |

| 函数                                     | 描述                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| fn:contains(string, substring)           | 如果参数string中包含参数substring，返回true                  |
| fn:containsIgnoreCase(string, substring) | 如果参数string中包含参数substring（忽略大小写），返回true    |
| fn:endsWith(string, suffix)              | 如果参数 string 以参数suffix结尾，返回true                   |
| fn:escapeXml(string)                     | 将有特殊意义的XML (和HTML)转换为对应的XML character entity code，并返回 |
| fn:indexOf(string, substring)            | 返回参数substring在参数string中第一次出现的位置              |
| fn:join(array, separator)                | 将一个给定的数组array用给定的间隔符separator串在一起，组成一个新的字符串并返回。 |
| fn:length(item)                          | 返回参数item中包含元素的数量。参数Item类型是数组、collection或者String。如果是String类型,返回值是String中的字符数。 |
| fn:replace(string, before, after)        | 返回一个String对象。用参数after字符串替换参数string中所有出现参数before字符串的地方，并返回替换后的结果 |
| fn:split(string, separator)              | 返回一个数组，以参数separator 为分割符分割参数string，分割后的每一部分就是数组的一个元素 |
| fn:startsWith(string, prefix)            | 如果参数string以参数prefix开头，返回true                     |
| fn:substring(string, begin, end)         | 返回参数string部分字符串, 从参数begin开始到参数end位置，包括end位置的字符 |
| fn:substringAfter(string, substring)     | 返回参数substring在参数string中后面的那一部分字符串          |
| fn:substringBefore(string, substring)    | 返回参数substring在参数string中前面的那一部分字符串          |
| fn:toLowerCase(string)                   | 将参数string所有的字符变为小写，并将其返回                   |
| fn:toUpperCase(string)                   | 将参数string所有的字符变为大写，并将其返回                   |
| fn:trim(string)                          | 去除参数string 首尾的空格，并将其返回                        |

在jsp页面里面最好不要出现java代码块，但是那个`<base href="<%=basePath%>">`我一直有点头疼，最近终于找到替换方案了，如下：

```jsp
<base href="${pageContext.request.scheme}://${pageContext.request.serverName}:${pageContext.request.serverPort}${pageContext.request.contextPath}/">
```

