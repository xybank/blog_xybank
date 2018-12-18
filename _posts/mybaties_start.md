---
title: mybaties总结
date: 2018-12-18 16:54:00 
tags: [mybaties] 
categories: mybaties
description: 记录mybaties的使用
---

##### 为什么要使用mybaties

由于jdbc有以下一些问题：

1、数据库连接创建、释放频繁造成系统资源浪费，从而影响系统性能。如果使用数据库连接池可解决此问题。
2、Sql语句在代码中硬编码，造成代码不易维护，实际应用中sql变化的可能较大，sql变动需要改变java代码。
3、使用preparedStatement向占有位符号传参数存在硬编码，因为sql语句的where条件不一定，可能多也可能少，修改sql还要修改代码，系统不易维护。
4、对结果集解析存在硬编码（查询列名），sql变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成pojo对象解析比较方便。

使用mybaties可以比较好的解决上面的问题

#####mybaties原理

![ybaties_0](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/mybaties_01.png)

mybatis配置
1、SqlMapConfig.xml，此文件作为mybatis的全局配置文件，配置了mybatis的运行环境等信息。
2、mapper.xml文件即sql映射文件，文件中配置了操作数据库的sql语句。此文件需要在SqlMapConfig.xml中加载。
通过mybatis环境等配置信息构造SqlSessionFactory即会话工厂
3、由会话工厂创建sqlSession即会话，操作数据库需要通过sqlSession进行。
4、mybatis底层自定义了Executor执行器接口操作数据库，Executor接口有两个实现，一个是基本执行器、一个是缓存执行器。
5、Mapped Statement也是mybatis一个底层封装对象，它包装了mybatis配置信息及sql映射信息等。mapper.xml文件中一个sql对应一个Mapped Statement对象，sql的id即是Mapped statement的id。
6、Mapped Statement对sql执行输入参数进行定义，包括HashMap、基本类型、pojo，Executor通过Mapped Statement在执行sql前将输入的java对象映射至sql中，输入参数映射就是jdbc编程中对preparedStatement设置参数。
7、Mapped Statement对sql执行输出结果进行定义，包括HashMap、基本类型、pojo，Executor通过Mapped Statement在执行sql后将输出结果映射至java对象中，输出结果映射过程相当于jdbc编程中对结果的解析处理过程。

##### 配置信息

mybaties的核心配置文件是sqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!-- 和spring整合后 environments配置将废除 -->
	<environments default="development">
		<environment id="development">
			<!-- 使用jdbc事务管理 -->
			<transactionManager type="JDBC" />
			<!-- 数据库连接池 -->
			<dataSource type="POOLED">
				<property name="driver" value="com.mysql.jdbc.Driver" />
				<property name="url"
					value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8" />
				<property name="username" value="root" />
				<property name="password" value="root" />
			</dataSource>
		</environment>
	</environments>
	
	<!-- 告诉Map的位置 -->
	<mappers>
	    <mapper resource="sqlmap/User.xml"/>
	</mappers>
</configuration>

```

所有的sql语句都写到配置文件里面去了

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 写sql语句 -->
<mapper namespace="test">
    <!-- 通过id查询用户 -->
    <select id="findUserById" parameterType="Integer" resultType="cn.itcast.mybatis.po.User" >
        select * from user where id = #{v}
    </select>
    <!-- 通过username模糊查询  parameterType表示参数类型 resultType表示返回类型-->
    <select id="findUserByUserName" parameterType="String" resultType="cn.itcast.mybatis.po.User" >
        <!-- select * from user where username like '%${value}%' 代替 这个时候一定要写value ${value}是连接符-->
        select * from user where username like "%"#{value}"%"
    </select>
</mapper>
```

测试

```java
package junit;

import java.io.InputStream;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import cn.itcast.mybatis.po.User;

public class MybatiesTest {
	//根据id查询
	public void testMybaties() throws Exception {
		//加载核心配置文件
		String resource = "sqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行Sql语句 10表示传到sql语句里面的参数
		User user = sqlSession.selectOne("test.findUserById", 10);
		System.out.println(user);
			
	}
	
    //模糊查询
	@Test
	public void testMybatiesByName() throws Exception {
		//加载核心配置文件
		String resource = "sqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		//创建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		//创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		//执行Sql语句
		List<User> users = sqlSession.selectList("test.findUserByUserName", "王");
		for (User user : users) {
			System.out.println(user);
		}
	}
}

```

注意：当返回是一个list的时候，使用selectList()方法查询，resultType表示的就是list的范型

###### 添加用户

```xml
<!-- 添加用户 -->
<insert id="insertUser" >
    <selectKey resultType="Integer" order="AFTER" keyProperty="id">
        select LAST_INSERT_ID()
    </selectKey>
    insert into user (username, birthday, sex, address) values
    (#{username}, #{birthday}, #{sex}, #{address})
</insert>
```

```java
@Test
public void testInsertUser() throws Exception {
    //加载核心配置文件
    String resource = "sqlMapConfig.xml";
    InputStream in = Resources.getResourceAsStream(resource);
    //创建SqlSessionFactory
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
    //创建SqlSession
    SqlSession sqlSession = sqlSessionFactory.openSession();
    User user = new User("何炅", "女", new Date(), "阳光大道18号");
    //执行Sql语句
    int index = sqlSession.insert("test.insertUser", user);
    sqlSession.commit();
    System.out.println(user.getId());
   ｝
```

注意：

LAST_INSERT_ID()是mysql提供的函数，表示最近增长的的数据的id，如果是mysql order要选择AFTER， oracle选择BEFORE， keyProperty表示放到对应的类的哪个属性里面

###### 修改用户

```java
public void UpdateUserById() throws Exception {
	    //加载核心配置文件
	    String resource = "sqlMapConfig.xml";
	    InputStream in = Resources.getResourceAsStream(resource);
	    //创建SqlSessionFactory
	    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
	    //创建SqlSession
	    SqlSession sqlSession = sqlSessionFactory.openSession();
	    User user = new User(29, "何日火", "女", new Date(), "阳光大道18号");
	    //执行Sql语句
	    int index = sqlSession.update("test.updateUser", user);
	    sqlSession.commit();
	}
```

```xml
<!-- 更新用户 -->
	<update id="updateUser" parameterType="cn.itcast.mybatis.po.User">
	    update user
	    set username = #{username}, birthday = #{birthday}, sex = #{sex}
	    where id = #{id}
	</update>
```

###### 删除用户

```java
public void DelUserById() throws Exception {
	    //加载核心配置文件
	    String resource = "sqlMapConfig.xml";
	    InputStream in = Resources.getResourceAsStream(resource);
	    //创建SqlSessionFactory
	    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
	    //创建SqlSession
	    SqlSession sqlSession = sqlSessionFactory.openSession();
	    //执行Sql语句
	    sqlSession.delete("test.deleteUserById", 28);
	    sqlSession.commit();
	}
```

```xml
<!-- 删除用户 -->
	<delete id="deleteUserById" parameterType="Integer">
	    delete from user where id = #{vvv}
	</delete>
```

##### mybaties和hibernate的区别

1、Mybatis和hibernate不同，它不完全是一个ORM框架，因为MyBatis需要程序员自己编写Sql语句。

2、Mybatis学习门槛低，简单易学

3、程序员直接编写原生态sql，可严格控制sql执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，例如互联网软件、企业运营类软件等，因为这类软件需求变化频繁，但需求变化要求成果输出迅速。

4、mybatis无法做到数据库无关性，如果需要实现支持多种数据库的软件则需要自定义多套sql映射文件，工作量大。

总之，按照用户的需求在有限的资源环境下只要能做出维护性、扩展性良好的软件架构都是好架构，所以框架只有适合才是最好。

##### Mapper动态代理

原则：

1、Mapper.xml文件中的namespace与mapper接口的完整类路径相同。
2、Mapper接口方法名和Mapper.xml中定义的每个statement的id相同 
3、Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同
4、Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同

例如：

接口

```java
public interface UserMapper {
    
	public User findUserById(Integer id);
	
}
```

UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 写Sql语句   -->
<mapper namespace="com.itheima.mybatis.mapper.UserMapper">
	<!-- 通过ID查询一个用户 -->
	<select id="findUserById" parameterType="Integer" resultType="cn.itcast.mybatis.po.User">
		select * from user where id = #{v}
	</select>
</mapper>
```

测试：

```java
@Test
	public void testMapper() throws Exception {
		String resource = "sqlMapConfig.xml";
		InputStream in = Resources.getResourceAsStream(resource);
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
		SqlSession sqlSession = sqlSessionFactory.openSession();
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		User user = userMapper.findUserById(10);
		System.out.println(user);
	}
```

小结：

mybatis官方推荐使用mapper代理方法开发mapper接口，程序员不用编写mapper接口实现类，使用mapper代理方法时，输入参数可以使用pojo包装对象或map对象，保证dao的通用性。

##### 取别名

可以在sqlMapConfig.xml里面设置别名，这样写sql语句的xml文件时就不需要写完整类名了，用别名即可

```xml
<typeAliases>
        <typeAlias type="cn.itcast.mybatis.po.User" alias="User"/>
 </typeAliases>
```

这是将cn.itcast.mybatis.po.User明确指定为User

```xml
<typeAliases>
        <package name="cn.itcast.mybatis.po"/>
</typeAliases>
```

这是指定cn.itcast.mybatis.po这个包下面所有类名，例如cn.itcast.mybatis.po.User指定为user和User，一般最好指定的包名下面都是pojo类

##### 映射器

在mapper标签里面有三个属性，分别是resource、url、class 这三个属性只能出现一个，resource用法如下

```xml
<mappers>
	 <mapper resource="com/itheima/mybatis/mapper/UserMapper.xml" />
</mappers>
```

使用mapper接口类路径

```xml
<mapper class="cn.itcast.mybatis.mapper.UserMapper"/>
```

注意：此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。

url要用绝对路径，现在不用

使用package接口类路径

注册指定包下的所有mapper接口

```xml
<package name="cn.itcast.mybatis.mapper"/>
```

注意：此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。现在package用得多



