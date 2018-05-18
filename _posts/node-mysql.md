---
title: node连接MySql操作数据库
date: 2018-05-18 13:15:12
tags: [node,Mysql]
categories: 匡利金
description: nodejs创建连接数据库，进行增删改查操作
---

nodejs的教程，大多以操作mongodb或者mysql。我打算用MySQL主要是因为懒，其次接触的比较多。

## 准备工作
1.安装mysql数据库

* **这里笔者也比较懒，使用的*xammp（Apache+MySQL+PHP+PERL）*套件，里面自带mysql数据库，默认账号root密码空，使用比较方便，不需要直接写在就好**。

* 1.1 连接数据库
新建一个库（db_rytong）
新建一个员工表（employee ）

字段 | 字段名称
---|---
emp_id | 主键ID
emp_name | 名称
emp_sex | 性别
emp_age | 年龄
emp_tel | 手机号码
emp_job | 职位
address | 住址
email | 邮箱

2.安装nodejs[链接地址:https://nodejs.org/en/download/](https://nodejs.org/en/download/)

*在cmd命令框里输入npm -v 查看
出现版本信息说明安装成功!*

3.新建文件目录

*  例如在D盘目录node文件夹下新建mysql文件夹。
目录：D:\node\mysql\

* 执行 npm install mysqy 安装插件

到此准备工作基本就做完了

## 连接数据库

```
var mysql      = require('mysql');
var connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'root',
  password : '',
  database : 'db_rytong'
});
```

创建数据库连接。


## 操作数据(增删改查)
1.查询数据

```
var mysql  = require('mysql');  
var connection = mysql.createConnection({   host     : 'localhost',
  user     : 'root',
  password : '',
  database : 'db_rytong'
}); 

connection.connect();

var  sql = 'SELECT * FROM employee';
connection.query(sql,function (err, result) {
    if(err){
        console.log('[SELECT ERROR] - ',err.message);
        return;
    }
    console.log(result);
});
connection.end();
```

查询结果如下：

 emp_id |emp_name | emp_sex | emp_age |emp_tel |emp_job | email 
---|---|---|---|---|---|---|---
1 |小明 |男|22|1333333333|java工程师|xiaoming@163.com
2|小红 |nv|18|13333344444|美工|xiaohong@163.com1 |小明 |男|22|1333333333|java工程师|xiaohongmao@163.com
3 |小李 |男|22|1337777777|销售|xiaolizi@163.com
4 |小丽 |女|23|15555555555|财务|xiaolil@163.com
5 |小h花 |nv|32|17788889999|财务总监|xiaohhua@163.com

2.插入数据


```
var mysql  = require('mysql');  
var connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'root',
  password : '',
  database : 'db_rytong'
}); 

connection.connect();
 
var  addData= 'INSERT INTO employee(emp_id ,emp_name ,emp_sex ,emp_age,emp_tel ,emp_job  ,email ) VALUES(0,?,?,?,?,?,?)';
var  addSqlParams = ['薛玉博', '男',19,'13838383838','实习生','caidashen@qq.com'];
//增
connection.query(addData,addSqlParams,function (err, result) {
    if(err){
        console.log('[INSERT ERROR] - ',err.message);
        return;
    }
connection.end();
```

生成结果如下：
emp_id |emp_name | emp_sex | emp_age |emp_tel |emp_job | email 
---|---|---|---|---|---|---|---
6 |薛玉博 |男|19|13838383838|实习生|caidashen@qq.com


3.更新数据
```
var mysql  = require('mysql');  
var connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'root',
  password : '',
  database : 'db_rytong',
}); 

connection.connect();
 
var  upData= 'UPDATE employee SET emp_name =?,emp_sex =?,emp_age =?,emp_tel =?,emp_job =?,email=?  WHERE emp_id = ?';
var updataSqlParams = ['A薛玉博', '男',199,'13838383838','A实习生','Acaidashen@qq.com',6];
//改
connection.query(upData,updataSqlParams,function (err, result) {
    if(err){
        console.log('[INSERT ERROR] - ',err.message);
        return;
    }
connection.end();
```
结果如下：
emp_id |emp_name | emp_sex | emp_age |emp_tel |emp_job | email 
---|---|---|---|---|---|---|---
6 |A薛玉博 |男|199|13838383838|A实习生|AcaAidashen@qq.com


4.删除数据

```
var mysql  = require('mysql');  
var connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'root',
  password : '',
  database : 'db_rytong',
}); 

connection.connect();
 
var  delData = 'DELETE FROM employee where id=1';
connection.query(delSql,function (err, result) {
    if(err){
        console.log('[DELETE ERROR] - ',err.message);
        return;
    } 
connection.end();
```
数据库结果如下：
 emp_id |emp_name | emp_sex | emp_age |emp_tel |emp_job | email 
---|---|---|---|---|---|---|---

2|小红 |nv|18|13333344444|美工|xiaohong@163.com1 |小明 |男|22|1333333333|java工程师|xiaohongmao@163.com
3 |小李 |男|22|1337777777|销售|xiaolizi@163.com
4 |小丽 |女|23|15555555555|财务|xiaolil@163.com
5 |小h花 |nv|32|17788889999|财务总监|xiaohhua@163.com

**好了nodejs-mysql就到这里了，有兴趣的可以去尝试是以下**