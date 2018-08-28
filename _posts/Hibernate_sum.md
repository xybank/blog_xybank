---
title: hibernate框架配置详解
date: 2018-08-28 16:54:00 
tags: [hibernate] 
categories: 甘明阳
description: 详细介绍hibernate框架的配置和使用
---

hibernate主要封装java里面jdbc这部分，是jdbc代码的轻量级的对象封装，使用hibernate有很多好处：

1、降低了代码的耦合性，程序员只需要专注业务代码即可

2、换了数据库只需要改动配置文件即可

3、大大提高程序员的开发效率

4、支持分布式架构

hibernate核心思想是ORM(object relation mapping)，官方推荐先写domain层，再写配置文件，最后生成数据库，但是实际上都是先设计好数据库，然后写配置文件，最后再写domain层，这里也是按照实际顺序配的

数据库代码(使用的是mysql数据库，以employee为例):

```bash
CREATE TABLE `employee` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(64) NOT NULL,
  `email` varchar(64) NOT NULL,
  `hiredate` date NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
```

我们在这张表里面设计了4列，分别是主键id（int类型），name（varchar类型），email（varchar类型），hiredate（date类型），那么我们的domain类的结构也要设计成一样

```java
package com.ganmy.doman;
//这个pojo对象应当按照规范序列号，目的是可以唯一标识该对象,同时可以在网络和文件上传输
public class Employee implements java.io.Serializable{
	
	//添加序列版本号
	private static final long serialVersionUID = 1L;
	private Integer id;
	private String name;
	private String email;
	private java.util.Date hiredate;
	
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
```

接下来是hibernate与数据库的连接配置文件hibernate.cfg.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
	"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
	"http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
       <!-- 配置使用的Driver -->
       <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
       <!-- 连接数据库的，设置编码为utf-8 -->
       <property name="connection.url">
            <![CDATA[jdbc:mysql://localhost:3306/users?useUnicode=true&characterEncoding=utf8]]>
       </property>
       <!-- 用户名 -->
       <property name="connection.username">root</property>
       <!-- 密码 -->
       <property name="connection.password">root</property>
       <!-- 配置dialect 方言  告诉hibernate使用的mysql数据库-->
       <property name="dialect">org.hibernate.dialect.MySQLDialect</property>
       <!-- 显示出对应的sql语句 -->
       <property name="show_sql">true</property>
       <!-- 优化sql语句格式，会自动换行，方便程序员阅读 -->
       <property name="format_sql">true</property>
        <!--create表示mysql如果没有该表则创建，如果有就先删掉再创建,update如果表结构没有改变则会不会删表
         就是没有添加新记录就不会创建-->
       <!--<property name="hbm2ddl.auto">create</property>-->
       <!-- 加载关系对象映射文件 com/ganmy/doman/指的是这个文件所在的包-->
       <mapping resource="com/ganmy/doman/Employee.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```

然后是对象关系映射文件，mvc模型的设计理念是一张表对应一个类，表里面一行对应一个对象，hibernate也是这样设计的，实现对象与数据库表的映射关系的核心文件是类名.hbm.xml，一般这个类名和表名一样，只是首字母大写，我们的表名是employee，那么对应的文件就是Employee.hbm.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--该文件要清楚地表述出 类 和 表 的对应关系-->
<!DOCTYPE hibernate-mapping PUBLIC
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
	
<hibernate-mapping package="com.ganmy.doman">
    <class name="Employee" table="employee">
	    <!-- id元素用于指定主键属性 -->
	    <id name="id" column="id" type="java.lang.Integer">
	        <!-- generator用于指定主键值生成的策略  使用increment自增长策略-->
	        <generator class="increment"></generator>
	    </id>
	    <!-- 除了主键的其他属性 property里面的name表示类的属性名，type表示这个属性的类型
         ， column里面的name表示表对应的一列，一般设计的时候数据库的结构和类的属性设计一样-->
	    <property name="name" type="java.lang.String">
	        <column name="name" not-null="false" />
	    </property>
	    <property name="email" type="java.lang.String">
	        <column name="email" not-null="false" />
	    </property>
	    <property name="hiredate" type="java.util.Date">
	        <column name="hiredate" not-null="false" />
	    </property>
    </class>
</hibernate-mapping>
```

最后是我们的调用，主要是main方法的代码：

```java
	public static void main(String[] args) {
		//将configuration configuration主要用来读取配置文件 默认找hibernate.cfg.xml文件 完成初始化
		Configuration configuration = new Configuration().configure();
		//创建sessionfactory 是一个重量级的接口
		SessionFactory sessionFactory = configuration.buildSessionFactory();
		//创建一个和数据库的会话
		Session session = sessionFactory.openSession();
		//对于hibernate而言，做增删改的时候要使用事务提交
		Transaction transaction = session.beginTransaction();
		//创建一个雇员对象
		Employee employee = new Employee();
		employee.setName("zhangsan");
		employee.setEmail("1556254@123.com");
		employee.setHiredate(new Date());
		//保存
		session.save(employee);
		//提交事务
		transaction.commit();
		session.clear();

	}
```

这里说明一下：

1、sessionFactory是一个重量级的接口，会占用很多内存，我们要保证一个数据库只对应一个sessionFactory

2、session指的是代码和数据库之间的会话，不是我们javaweb里面的session

3、hibernate设计者设计在对表进行增删改操作的时候要使用事务提交，如果不使用事务提交将无法生效，也不报错，下面是具体的优化，由于sessionFactory是一个重量级的接口，会占用很多内存，我们要保证一个数据库只对应一个sessionFactory，使用静态代码块和单例模式来加载sessionFactory的创建，将sessionFactory封装到一个类里面

```java
final public class MySessionFactory {
	//一个sessionFactory对应一个数据库，sessionFactory是一个接口
	private static SessionFactory sessionFactory=null;
	
	private MySessionFactory(){};
	
	static{
		sessionFactory = new Configuration().configure().buildSessionFactory();
	}
	
	public static SessionFactory getSessionFactory(){
		return sessionFactory;
	}

}
```

```java
    /**
	 * @param args
	 */
	public static void main(String[] args) {
		updateEmployee_roll();

	}
     /**
     * 使用回滚的事务提交    
     */
	private static void updateEmployee_roll() {
		//修改用户
		//获取会话
		Session session = MySessionFactory.getSessionFactory().openSession();
		Transaction ts = null;
		try{
			ts = session.beginTransaction();
			//获取要修改的用户，然后修改,可以通过主键属性获取该对象实例，和表的记录对应
			Employee emp=(Employee)session.load(Employee.class, 2);
			emp.setName("甘明阳11");
			emp.setEmail("123@11");
			int i = 9/0;
			ts.commit();
			session.close();
		   }catch(Exception e){
			   if(ts != null){
				   ts.rollback();
			   }
			   throw new RuntimeException(e.getMessage());
		   }finally{
			   //关闭session
			   if(session!=null&&session.isOpen()){
				   session.close();
			   }
			   
		   }
	}
```

使用事务模版写最安全，即使报错不会提交到数据库，可以保证数据的完整性，而且能快速定位到错误。

##### session的获取

session的获取方式有两种，分别是openSession()和getCurrentSession()，特点：

1、openSession()是获取一个新的session

2、getCurrentSession()获取和当前线程绑定的线程，换言之，在同一个线程中，我们获取的session是同一个session，这样可以利于事务的控制

3、如果希望使用getCurrentSession()，需要配置hibernate.cfg.xml，添加：

```bash
//thread代表session可以和当前线程绑定
<property name="current_session_context_class">thread</property>
```

根据实际应用选择使用那个session，如果在同一个线程中，保证要使用同一个session就使用getCurrentSession()，如果在一个线程中要使用不同的session就使用openSession()

4、通过getCurrentSession()获取的session在事务提交以后会自动关闭，通过openSession()获取的session必须手动关闭

5、如果是通过getCurrentSession()获取的session在进行查询的时候也要以事务的方式提交，而且在commit以后，对象获取属性会报错could not initialize proxy - no Session，要在commit之前获取属性

##### 全局事务和本地事务

假如我们使用的数据库只有一个，那么控制这个数据库的事务就是本地事务，假如我们连接多个数据库，比如银行的转账系统，从工行转账到农行，那么控制这个跨数据库的事务就被成为全局事务(jta)，全局事务的价值更高

##### session获取对象的两种方法

session获取对象的两种方法分别是get()和load()，他们的区别是：

1、查询没有的数据返回不同，例如一个表只有10行数据，我们去第100行的数据，那么使用get()会返回null，而使用load()会报错ObjectNotFoundExpcetion

2、使用get查询数据会先到session缓存去查然后去二级缓存去查，如果没有立即向数据库发出sql语句，查询到了不会发sql语句。

使用load()去查询，也是先到session缓存去查然后去二级缓存去查，如果没有找到就返回一个代理对象，不会立即向数据库去查，等到后面使用这个代理对象操作的时候，才到DB中去查，这个现象我们称为lazy load(懒加载)，如果用户后来真的要用到这个对象，那么load()回到数据库去查，然后把查询的的结果存放到二级缓存里面，下一次再用这个sql语句去查询还是按照一级缓存、二级缓存的顺序去找，这个时候二级缓存已经有这个数据就不会到数据库去查了，如果二级缓存的命中率很高就会将这个结果放到一级缓存里面。

3、通过修改配置文件可以取消懒加载，在`*`.hbm.xml里面的class标签添加一个属性laze="false"就行了

4、如果确定数据库一定有这个对象就使用load()，效率比较高，如果不确定就使用get()，防止报错

##### 一级缓存和二级缓存

一级缓存就是sessionfactory的缓存，又叫session级缓存，二级缓存是介于文件和内存之间的缓存，一级缓存不需要配置就可以用，二级缓存需要配置启用才能用，缓存的机制可以减少对数据库的频繁访问，由于hibernate的缓存机制实现的很好，使用hibernate可以优化对数据库查询的次数

##### 线程局部变量模式

一般我们的变量范围是一个函数域，但是其实变量是可以和一个线程绑定的，这就是线程局部变量模式，还是以session为例

```java
package com.ganmy.util;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
	
	private static SessionFactory sessionFactory = null;
	
	//threadLocal是本地线程
	private static ThreadLocal<Session> threadLocal = new ThreadLocal<Session>();
	
	//将构造方法私有化
	private HibernateUtil(){};
	
	static{
		sessionFactory = new Configuration().configure().buildSessionFactory();
	}
	
	//获取全新的session
	public static Session OpenSession(){
		return sessionFactory.openSession();
	}
	
	//获取与当前线程绑定的session
	public static Session getCurrentSession(){
		Session session = threadLocal.get();
		if(session == null){
			session = sessionFactory.openSession();
			threadLocal.set(session);
		}
		return session;
	}		
}
```

将session放到本地线程threadLocal里面，以后可以直接从threadLocal获取，这样写就不需要配置thread了。

##### Query接口

在hibernate里面的查询使用hql语句，使用hibernate我们只需要写hql语句查询，不需要分数据库，因为hibernate会根据不同的数据库将hql语句翻译成对应的sql语句，例如：

```java
public static void main(String[] args) {
		Session session = HibernateUtil.getCurrentSession();
		Transaction ts = null;
		try{
			ts = session.beginTransaction();
            //使用hql语句查询，Employee表示对应的domain对象，id表示domain里面的id属性，不是数据库表里面的列名称
            //where后面的条件可以是类的属性名，也可以是表的字段，最好使用类的属性名称
			Query query = session.createQuery("from Employee where id = 100");
			//通过list方法获取结果，这个list会自动通过反射机制封装成对应的domain对象，不需要进行二次封装
			List<Employee> list = query.list();
			for(Employee e:list){
				System.out.println(e.getName()+e.getEmail());
			}
		}catch(Exception e){
			if(ts != null){
                ts.rollback();
			}
		}
	}
```

##### 使用逆向工程生成配置文件

一、使用数据库浏览器连接数据库

1、创建数据库表

2、创建java项目

3、通过myeclipse的数据库浏览器连接到mysql数据库

打开数据库浏览器 ->windows -> open persective -> Myeclipse Database Explorer->在DB Brower点击右键->

选择new，按照如图所示填写

![Q图片2018082716190](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/hibernate_00.png)

点击Test Driver弹框success表示连接成功

然后在打开右边的项目，找到table，右键，Edit data就可以了

注意mysql5.6的版本不能用5.0的jar包连接，会报错，要用支持5.6的jar包需要使用5.1.25版本的驱动，而且貌似数据库浏览器好像有缓存的样子，我换了一个高版本的jar包还是报错，然后我以为是jar版本不对，陆陆续续在网上找了很多版本的jar包，但是无一例外全部报错，我真的被搞的头都大了，搞了一天没找到原因，后来我思路一转，是不是数据库的问题不是jar包的问题呢，连接我云服务器的mysql，结果可以，然后再连接本地的mysql也都可以了，真的是一脸懵逼的解决了问题。

二、引入hibernate开发包

项目右键->myeclipse->add hibernate capabilities，按照如图所示填写

![Q图片2018082811070](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/hibernate_02.png)

点击Next

![Q图片2018082811115](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/hibernate_03.png)

点击Next

![Q图片2018082811161](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/hibernate_04.png)

点击Next

![Q图片2018082811193](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/hibernate_05.png)

如果希望把hibernate开发包升级，我们可以重新引入包，到文件路径把自动引入的jar包删掉，然后将我们自己的jar包引入

然后我们到数据库浏览器这一边，右键点击表，选择hibernate reverse engineering![Q图片2018082811425](https://github.com/ganmyxh/ganmyxh.io/blob/master/img/hibernate_06.jpg?raw=true)

点击browse选择包，将domian对象所在的包引入，下一步

、![Q图片2018082813325](https://raw.githubusercontent.com/ganmyxh/ganmyxh.io/master/img/hibernate_07.png)

然后下一步下一步点finally就可以了

最后：感觉hibernate真的大大提高了程序员的效率，不用写jdbc代码了，是一个值得学习的框架

