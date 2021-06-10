### MySQL

#### 一、数据库

> 什么是数据库

​	数据库是一个以某种有组织的方式存储的数据集合，也就是说，**数据库是数据的集合** 。

​	我们通常会用数据库来代表我们使用的数据库软件（MySQL），确切地说，数据库软件是一个数据库管理系统，管理数据的集合，也就是管理数据库。

> 数据库的术语

* 数据库：保存有组织的数据的容器
* 表：某种特定类型数据的结构化清单，由一个或多个列组成
* 模式：关于数据库和表的布局及特性的信息
* 列：表中的一个字段
* 数据类型：表中允许存放的数据的类型，每一列都有其对应的数据类型
* 行：表中的一个记录
* 主键：能够区别于其他行的一列或一组列

#### 二、SQL

> 什么是SQL

​	SQL（Structured Query Language）是结构化查询语言，用来操作数据库的语言,每一条SQL语句以“;“分号结束，SQL不区分大小写，通常关键字用大写，其他用小写。

> 简单SQL

* 选择数据库：USE 数据库名; 如 use students;
* 显示数据库：SHOW DATABASES;
* 显示数据表：SHOW TABLES;
* 显示表中的列：SHOW COLUMS FROM 表名; 如 SHOW COLUMS FROM students;
* 显示服务器状态信息：SHOW STATUS;
* 显示创建库或表语句：SHOW CREATE DATABASE;SHOW CREATE TABLE;
* 显示用户权限：SHOW GRANTS;
* 显示服务器错误或警告信息：SHOW ERRORS;SHOW WARNINGS;

#### 三、检索数据

> SELECT语句

* 检索单列：SELECT 列名 FROM 表名; 如：SELECT name FROM students;
* 检索多列：SELECT 列名,列名 FROM 表名; 如：SELECT name,age FROM students;
* 检索所有列：SELECT * FROM 表名; 如：SELECT * FROM students；
* 检索不同的行：SELECT DISTINCT 列名 FROM 表名; 
* 限制结果：SELECT 列名 FROM 表名 LIMIT 行数,起始行;同SELECT 列名 FROM 表名LIMIT 行数 OFFSET 起始行;*MySQL检索出来的行从第0行开始*  
* 使用完全限定名：SELECT 表名.列名 FROM 库名.表名;


> ORDER BY子句

* 升序（默认）：SELECT name FROM students ORDER BY name;
* 降序：SELECT name FROM students ORDER BY name DESC;
* 排序多个列：SELECT name,age FROM students ORDER BY age DESC,name;先对age进行排序，当age相同时再对name排序

> WHERE子句

* 单值检查

  | 操作符  | 说明   | SQL                                      |
  | ---- | ---- | ---------------------------------------- |
  |      | 等于   | SELECT name FROM students WHERE age=10;  |
  | <>   | 不等于  | SELECT name FROM students WHERE age<>10; |
  | !=   | 不等于  | SELECT name FROM students WHERE age!=10; |
  | >    | 大于   | SELECT name FROM students WHERE age>10;  |
  | >=   | 大于等于 | SELECT name FROM students WHERE age>=10; |
  | <    | 小于   | SELECT name FROM students WHERE age<10;  |
  | <=   | 小于等于 | SELECT name FROM students WHERE age<=10; |

* 范围检查：SELECT name FROM students WHERE age BETWEEN 10 AND 18;

* 空值检查：SELECT name FROM students WHERE age IS NULL | IS NOT NULL;             

* AND操作符：SELECT name FROM students WHERE age=18 AND scope=90;

* OR操作符：SELECT name FROM students WHERE age=18 OR scope=90；


  *优先级：AND操作符 > OR操作符* 

* IN操作符：SELECT name FROM students WHERE age IN (18,20);

* NOT操作符：SELECT name FORM students WHERE age NOT IN (18,20);

* LIKE操作符

  * 通配符
    * 百分号%：任意字符出现任意次数，SELECT name FROM students WHERE name="张%";
    * 下划线_：匹配一个字符，SELECT name FROM students WHERE name="张__";