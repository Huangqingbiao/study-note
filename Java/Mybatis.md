## Mybatis

### 1、Mybatis框架简介

​	Mybatis是一个开源的数据持久层框架，内部封装了通过JDBC访问数据库的操作，支持普通SQL查询、存储过程和高级映射，其核心思想是将程序中的大量SQL语句剥离出来，配置在配置文件中，实现SQL的灵活配置。

```
存储过程（Stored Procedure）是一组为了完成特定功能的SQL 语句集，经编译后存储在数据库中。用户通过指定存储过程的名字并给出参数（如果该存储过程带有参数）来执行它。
```

### 2、什么是ORM

对象/关系映射（Object/Relational Mapping, ORM）是一种数据持久化技术。它在对象模型和关系型数据库之间建立起对应关系，并且提供了一种机制，通过JavaBean对象去操作数据库表中的数据。

### 3、Mybatis核心配置文件

Mybatis核心配置文件主要用于配置数据库连接和Mybatis运行时所需的各种特性，包含了设置和影响Mybatis行为的属性。

### 4、Mybatis的核心对象

```
SqlSessionFactoryBuilder
通过读取配置信息或类来构建一个SqlSessionFactory
```

```
SqlSessionFactory
通过openSession（）方法来获取SQLSession实例
```

```
SqlSession
用于执行持久化操作，类似JDBC中的Connection，提供了面向数据库执行SQL命令所需的所有方法，可以通过Sqlsession实例直接运行映射的SQL语句。
```

