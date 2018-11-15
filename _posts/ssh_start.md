---
title: ssh框架整合
date: 2018-11-15 16:54:00 
tags: [ssh] 
categories: 甘明阳
description: 将struts、hibernate和spring三大框架整合起来
---

这段时间主要在看ssh，发现有了框架之后开发效率有了飞快的提升，尽管struts现在有的人很少了，但是ssh的思想依然值得学习，由于使用ssh整合，hibernate的hibernate.cfg.xml文件被spring接管，sessionFaction在spring配置文件里面配置即可。

先说一下hibernate和spring的整合

首先创建domain对象Employee

```java
package com.ganmy.domain;

import java.util.Date;

public class Employee {
	
	private Integer id;
	private String name;
	private String email;
	private java.util.Date hiredate;
	private Float salary;
	
	public Employee(){};
	
	public Employee(String name, String email, Date hiredate, Float salary) {
		this.name = name;
		this.email = email;
		this.hiredate = hiredate;
		this.salary = salary;
	}
	public Employee(Integer id, String name, String email, Date hiredate,Float salary) {
		this.id = id;
		this.name = name;
		this.email = email;
		this.hiredate = hiredate;
		this.salary = salary;
	}
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public java.util.Date getHiredate() {
		return hiredate;
	}
	public void setHiredate(java.util.Date hiredate) {
		this.hiredate = hiredate;
	}
	public Float getSalary() {
		return salary;
	}
	public void setSalary(Float salary) {
		this.salary = salary;
	}
}
```

配置对应的hbm文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--该文件要清楚地表述出 类 和 表 的对应关系-->
<!DOCTYPE hibernate-mapping PUBLIC
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
	
<hibernate-mapping package="com.ganmy.domain">
    <class name="Employee" table="employee" lazy="false">
	    <!-- id元素用于指定主键属性 -->
	    <id name="id" column="id" type="java.lang.Integer">
	        <!-- generator用于指定主键值生成的策略  使用native策略 "assigned策略表示可以指定id"  -->
	        <generator class="increment"></generator>
	    </id>
	    <!-- 除了主键的其他属性 -->
	    <property name="name" type="java.lang.String">
	        <column name="name" not-null="false" length="64"/>
	    </property>
	    <property name="email" type="java.lang.String">
	        <column name="email" not-null="false" length="64" />
	    </property>
	    <property name="hiredate" type="java.util.Date">
	        <column name="hiredate" not-null="false" />
	    </property>
	    <property name="salary" type="java.lang.Float">
	        <column name="salary" not-null="false" />
	    </property>
    </class>
</hibernate-mapping>
```

在数据库中创建对应的数据库表，创建service和serviceinterface包，创建对应的接口和实现接口的service类

```java
package com.ganmy.service.interfaces;

import java.util.List;

import com.ganmy.domain.Employee;

public interface EmployeeServiceInt {
	
	//声明保存雇员的方法
	public void addEmployee(Employee e);
	
	//显示所有雇员
    public List<Employee> showEmployee();
    
    //更新雇员
    public void updateEmployee(Employee e);
    
    //根据id号删除雇员,由于int和float都实现了Serializable这个接口，使用Serializable更加通用
    public void deleteEmployee(java.io.Serializable id);
}

```

```java
package com.ganmy.service.imp;

import java.io.Serializable;
import java.util.List;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.springframework.transaction.annotation.Transactional;

import com.ganmy.domain.Employee;
import com.ganmy.service.interfaces.EmployeeServiceInt;

//这里配置注解@Transactional，用处是让spring的事务管理器接管该service的所有事务
//这个注解放在类上面就对整个类生效，放在方法上面就对整个方法生效
@Transactional
public class EmployeeService implements EmployeeServiceInt{
	
	private SessionFactory sessionFactory;

	public SessionFactory getSessionFactory() {
		return sessionFactory;
	}

	public void setSessionFactory(SessionFactory sessionFactory) {
		this.sessionFactory = sessionFactory;
	}

	public void addEmployee(Employee e) {
//		Session session = sessionFactory.openSession();
//		Transaction tx =  (Transaction) session.beginTransaction();
//		session.save(e);
//		tx.commit();
		sessionFactory.getCurrentSession().save(e);
	}

	public void deleteEmployee(Serializable id) {
		
	}

	public List<Employee> showEmployee() {
		return null;
	}

	public void updateEmployee(Employee e) {
		
	}
}
```



到这里基本工作就完成了，由于hibernate.cfg.xml被整合到spring里面，接下来配置ApplicationContext.xml文件



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:context="http://www.springframework.org/schema/context"
		xmlns:tx="http://www.springframework.org/schema/tx"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
				http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd
				http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd"
				>
   <!-- 配置测试类 -->
   <bean id="testService" class="com.ganmy.test.TestService">
       <property name="name">
           <value>明阳</value>
       </property>
   </bean>
   <!-- 配置数据源 -->
   <!-- 配置databaseSource -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
	    <property name="url" value="jdbc:mysql://localhost:3306/users?useUnicode=true&amp;characterEncoding=utf-8"/>
	    <property name="username" value="root"/>
	    <property name="password" value="root"/>
	    <!-- 连接池启动时的初始值 -->
		<property name="initialSize" value="3"/>
		<!-- 连接池的最大值 -->
		<property name="maxActive" value="500"/>
	    <!-- 最大空闲值.当经过一个高峰时间后，连接池可以慢慢将已经用不到的连接慢慢释放一部分，一直减少到maxIdle为止 -->
		<property name="maxIdle" value="2"/>
		<!--  最小空闲值.当空闲的连接数少于阀值时，连接池就会预申请去一些连接，以免洪峰来时来不及申请 -->
		<property name="minIdle" value="1"/>
	</bean>
	<!-- 配置sessionFactory -->
	<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <!-- 设置数据元 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 接管hibernate里面的对象关系映射文件 -->
        <property name="mappingResources">
		    <list>
		      <value>com/ganmy/domain/Employee.hbm.xml</value>
		    </list>
	    </property>
	    <property name="hibernateProperties">
		     <value>
		        	hibernate.dialect=org.hibernate.dialect.MySQLDialect
		        	hibernate.hbm2ddl.auto=update
					hibernate.show_sql=true
					hibernate.format_sql=true
					hibernate.cache.use_second_level_cache=true
        	        hibernate.cache.provider_class=org.hibernate.cache.EhCacheProvider
        	        hibernate.generate_statistics=true	 
		     </value>
	    </property>
    </bean>
    <!-- 配置service -->
    <bean id="employeeService" class="com.ganmy.service.imp.EmployeeService">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>
    <!-- 配置事务管理器 ,统一管理sessionFactory的事务-->
    <bean id="txManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
  	    <property name="sessionFactory" ref="sessionFactory"/>
	</bean>
	<!-- 启用事务的注解 -->
	<tx:annotation-driven transaction-manager="txManager"/>
</beans>
```

#####数据源的概念：

1、初始值，连接池默认开始的连接数量，可以瞬间使用，不需要建立驱动的连接

2、最大值，是对数据库的保护机制，比如最大值为500时，当第501个连接来了就会等待，如果前面的连接释放了才会去连接

3、最大空闲值，当经过一段时间后，连接池会慢慢释放一部分连接，比如上面的配置会释放到2个连接

4、最小空闲值，如上面，一共500个连接，当用499个连接被使用的时候，连接池会去创建连接，防止来不及申请

##### 测试：

```java
package com.ganmy.test;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.ganmy.domain.Employee;
import com.ganmy.service.interfaces.EmployeeServiceInt;

public class Test {
	public static void main(String[] args) {
		ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
		Employee emp = new Employee("aa111", "1535628743@qq.com", new java.util.Date(),23.456f);
		EmployeeServiceInt es = (EmployeeServiceInt) ac.getBean("employeeService");
		es.addEmployee(emp);
		
	}

}
```

配置struts

struts-config.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE struts-config PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 1.3//EN" "http://struts.apache.org/dtds/struts-config_1_3.dtd">
<struts-config>
    <form-beans>
        <form-bean name="employeeForm" type="com.ganmy.web.forms.EmployeeForm"/>
    </form-beans>
    <action-mappings>
        <action path="/login" parameter="flag" name="employeeForm">
            <forward name="ok" path="/WEB-INF/MainFrame.jsp" />
            <forward name="err" path="/WEB-INF/login.jsp" />
        </action>
    </action-mappings>
    <!-- 配置代理请求处理 DelegatingRequestProcessor ,它的用处是可以让spring来获取strut的action信息和配置action的一些属性-->
	<controller>
 	<set-property property="processorClass" value="org.springframework.web.struts.DelegatingRequestProcessor"/>
	</controller> 
</struts-config>
```

web.xml

```xml
 <!-- 相对src目录寻找的 ，tomcat启动的时候会去实例化spring容器，applicationContext.xml配置的bean对象都实例化了-->
	<context-param>
	   <param-name>contextConfigLocation</param-name>
	   <param-value>classpath:applicationContext.xml</param-value>
	</context-param>
	<!-- 对Spring容器进行实例化 -->
	<listener>
	      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
```

applicationContext.xml

```xml
<!-- 配置action -->
	<bean name="/login" class="com.ganmy.web.action.LoginAction" scope="prototype">
	    <property name="employeeServiceInt" ref="employeeService"/>
	</bean>
```

解决中文乱码问题：

1、自己配置过滤器

创建com.ganmy.web.filter包，创建一个servlet名为MyFilter实现javax.servlet.Filter这个接口

内容：

```java
arg0.setCharacterEncoding("utf-8");//设置接收编码
arg2.doFilter(arg0, arg1);//必须要有，如果没有会停止前进
```

在web.xml里面添加

```xml
    <filter>
      <filter-name>Myfilter</filter-name>
      <filter-class>com.ganmy.web.filter.Myfilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>Myfilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

2、使用spring提供的过滤器

```xml
<filter>
		<filter-name>encoding</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encoding</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```

##### 使用注解的方式配置属性

分三步：

1、配置bean，不用注入属性

```java
<bean id="employeeService" class="com.ganmy.service.imp.EmployeeService"/>
```

3、在对应的属性sessionFactory上方添加注解

```java
//当我们给某个属性增加了resource后,spring就会启动byName的方式注入属性值
@Resource
private SessionFactory sessionFactory;
```

3、启用注解扫描

```xml
<context:annotation-config/>
```

是使用byName的方式匹配的，由于创建对象的时候这个属性指向null，于是就在配置文件里面的bean里面找一个id和这个name相同的对象指向给这个属性，这就是byName的方式

##### 懒加载的问题

ssh整合的时候如何解决懒加载的问题，比如有多张表关联，对象对于外部关联的表会遇到懒加载的情况，解决方案：

1、显示初始化

```xml
//显示初始化，防止懒加载
Hibernate.initialize(Department.class);
//但是要先打印才行
System.out.println(ee.getDepartment().getName());
```

2、在配置文件里面添加lazy="false"

但是这两种方法不好，不管jsp用不用都会向数据库查询，最好是让session的生命周期变长一点

3、spring专门提供的方法opensessioninview，需要在web文件中添加一个过滤器，原理是延缓了commit的时间

```xml
<!-- 配置opensessioninview解决懒加载,本质一个过滤器. -->
<filter>
        <filter-name>OpenSessionInViewFilter</filter-name>
        <filter-class>org.springframework.orm.hibernate3.support.OpenSessionInViewFilter</filter-class>
</filter>
<filter-mapping>
        <filter-name>OpenSessionInViewFilter</filter-name>
        <url-pattern>/*</url-pattern>
</filter-mapping>
```

该方法可以减少使用sql的次数，缺点是会延长和数据库保持的session的生命周期

##### 基础接口

通过在web层和业务层添加一层基础接口BasicService来提高代码的复用性，简单的说就是将所有service里面的通用的方法写到基础接口里面，然后通过继承的方式来获取这些方法，然后自己独特的方法就放到自己的接口里面，然后实现这个接口