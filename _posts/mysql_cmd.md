---
title: mysql基本操作
date: 2018-09-16
tags: [SQL]
categories: 陈志奇
description: 本文主要描述通过控制台来对mysql操作的基础知识
---

# 使用命令操作mysql数据库基础语句

> 如未安装mysql，请先安装配置好mysql，链接 [mysql安装教程](http://www.runoob.com/mysql/mysql-install.html)

## 启动和关闭mysql服务器
 - 启动服务 ：

   在你的C:\Windows\System32目录下找到 cmd  并且以管理员身份运行cmd
   
   输入net start 服务器名称
   注：服务器名称就是安装mysql的时候自己定义的。
``` mysql
net start MySQL57
```   
- 关闭服务
 
   输入net stop 服务器名称


``` mysql
net stop MySQL57
```
--------------------
## 链接Mysql
- 连接数据库 
 
   语法：mysql -u 用户名 -p  ( 回车后，输入安装时设置的密码 )

``` mysql
mysql -u root -p
```
 
 
- 退出登入（断开链接）
  
  语法：quit 或 exit
``` mysql
exit;
```
 
- 查看版本  
 ``` mysql 
 select version();
 ```

- 显示当前时间  
``` mysql 
select now();
```
- 远程链接
  
  语法：mysql  -h  ip地址  -u  用户名称 -p
  
``` mysql 
mysql  -h  192.168.13.23  -u  qiqi -p
```

### 数据库操作
 - 查看当前选择的数据库
 
``` mysql 
select databases;
```

 - 创建数据库

   语法：create database 数据库名; charset=utf8;
   
   eg：创建一个school数据库
   
``` mysql 
create database school charset=utf8;
```
 - 删除数据库
   
   语法：  drop database if exists 数据库名; 
   
``` mysql 
drop database if exists school;
```
- 切换数据库
  
   语法：  use 数据库名; 
   
``` mysql 
use school
```
--------------
## 表操作

- 查看当前数据库中所以表

``` mysql 
show tables;
```  

- 创建表
    
   语法：create  table 表名(列 类型);
     
``` mysql
create table class(id int auto_increment primary key,
                                        name varchar(20) not null,
                                        gender bit default 1);
create table student(id int auto_increment primary key,
                                        name varchar(20) not null,
                                        age int not null,
                                        gender bit default 1,
                                         classid int not null ,
                                         foreign key(classid) references class(id));
```  
- 删除表
    
  语法：drop table if exists 表名

``` mysql 
eg:如果存在class这张表则删除
drop table if exists class;
```

- 查看表结构
    
    格式：desc 表名;

``` mysql
eg:查看class的表结构    
desc class;
```

- 查看建表语句
  
   语法： show create table 表名;

``` mysql 
eg:查看class的建表语句
show create table class;
```

- 重命名表名
    
    语法： rename table 原表名 to 新表名；
    
``` mysql 
eg：将class重新命名为 newclass
rename table class to newclass;
```

- 修改表结构
  
 语法:

a、alter table 表名  add   列名  类型；

b、alter table 表名  drop  列名；

``` mysql
eg:给student 增加一个性别列
alter table student add sex char(2) default '男';
    
eg:给student删除性别列：
alter table student drop sex;
```
---------
## 数据操作
- 增加数据

a、全列插入:

``` mysql 
insert  into 表名 values(.....);
注意：主键列是自动增长，但是在全列插入时需要占位，通常使用0，插入成功以后以实际数据为准
```
b、缺省插入:

``` mysql 
insert  into  表名(字段) values(.....);
```    

c、同时插入多条数据:

``` mysql 
insert into 表名 values(allvalue1),(allvalue2)...(allvalue3);
```  

- 删除
  
  语法：delete from 表名 where 条件；
``` mysql 
eg:删除student中sex值为男
delete from student where sex = '男'；
```  

- 改
   
  语法：update 表名 set 列1=值1 ，列2=值2 where 条件；
 
```mysql 
eg:将id为1的student中sex值设为男
update student set sex = '男' where id = 1；
``` 

- 查
     
1、基本查询语法：
        select * from 表名；
        
``` mysql 
eg:查询所有学生
select * from student；
``` 
   2、消除重复行
   
``` mysql 
select distinct * from 表名 where 条件; 
```   

   3、条件查询
   
``` mysql 
   a、基本查询条件语法
   
    select * from 表名  where 条件;
   b、比较运算符：
                    等于  =
                    大于  >
                    小于  <
                    大雨等于 >=
                    小于等于 <=
                    不等于 != (<>)
                                     
   c、逻辑运算符
                  and——————并且
                   or——————或者
                  not——————非
                   
    d、范围查询  
     in  
     表示在一个非连续的范围内
     between -----and -----
     表示在一个联系的范围内
                   
     e、空判断  
        注意：null 与"" 是不同的   
        判断空是：is null 
            非空：is not null 
                   
      f、优先级
        小括号，  
        not 比较运算符，
        逻辑运算符，
        注：and比or优先级高，如果同时出现并希望先选or，需要结合（）使用。
                   
      g、模糊查询
         like   
         %任意多个字符， _表示一个任意字符。 
``` 

       
- 聚合（为了快速统计数据，提供了5个聚合函数）

 a、count(*)表示计算总函数，括号可以写*和列名
 
``` mysql 
eg:查询学生人数
select count(*) from student；
```

b、max（列） 表示求此列的最大值

``` mysql 
eg:查询学生最大年龄
select max(age) from student；
```                 
 c、min(列) 表示求此列的最小值
 
``` mysql 
eg: 查询学生最小年龄
select min(age) from student；
``` 
 
 d、sum(列) 表示求此列的和
 
``` mysql 
eg:查询学生年龄和
select sum(age) from student；
```  

 e、avg(列)表示求此列的平均值
 
``` mysql
eg:查询学生平均年龄
select max(age) from student；
```  

- 排序
                  
  语法：select * from 表名 order by 列1 [asc|desc]，列2 [asc|desc]
  
``` mysql
eg:按照学生年龄降序排列
select * from student order by age desc；
```    
               
- 分页

语法：select * from 表名 limit start,count     start 从0开始

``` mysql
eg:按照学生年龄降序排列
select * from student order by age desc；
``` 

## 关联查询
 基本语法： select 表1.列,表2.列，... from 表1 ( left | right | inner)  join 表2 on 表1.列 = 表2.列 ;        

``` mysql 
left | right | inner说明：
a、表A inner join 表B (显示表A与表B匹配的行的结果集)        
b、表A left join 表B  (显示表A与表B匹配的行以及表A未匹配的行的结果集)   
c、表A rightjoin 表B  (显示表A与表B匹配的行以及表B未匹配的行的结果集)   
``` 