## Spring IOC容器

###  1、简介

​	Spring IOC容器是一个管理Bean的容器，所有的IOC容器必须实现了BeanFactory接口，BeanFactory接口中定义了多个getBean()方法来向IOC容器中获取Bean。（可通过类型和名称获取Bean。

```
IOC中获取的Bean对象默认是单例的（isSingleton)
```

### 2、相关注解

##### 	@Configuration

```
被@configuration标注的类代表为是一个java配置类，相当于xml和properties文件。Spring会根据这个类来生成IOC容器去装配Bean。
```

##### 	@Bean（name="")

```
被@Bean注解标注的方法的返回值装配到IOC容器中，Bean的名称则是name的值，如果没有指定，则Bean的名称是方法名。
```

##### 	例子

```
@Configuration
public class IocConfig{
  @Bean(name = "user")
  public User user(){
    User user = new User();
    ...
    return user;
  }
}

最后通过 new AnnotationConfigApplicationContext(IocConfig.class)来加载配置类，将配置类中的Bean转配到IOC容器中。
```

##### 	@Component()

```
被@Component注解标注的类可以被扫描装配到IOC容器中。
```

##### 	@ComponentScan

```
@componentScan可以标明采用何种策略去扫描转配Bean
```

##### 	@Value

```
@Value可以给Bean的属性值赋值。
```

##### 	例子

```
@Component("user")
public class User{
  @value("1")
  private int id;
  ...
}
@ComponentScan（basePackages = {"需要扫描的报名",...}
|basePackageClasses = {类名.class,...},
excludeFilters = {@Filter(classes = {类名.class,...})},...)
public class IocConfig{
  //默认只会扫描当前包及其子包,支持正则表达式
}
```

#### 3、依赖注入（DI)

##### 	@Autowired

```
被@Autowired注解标注的对象会根据类型向IOC容器中获取Bean，被称为注入。相当于new了一个对象。
规则：首先会根据类型寻址Bean,如果对于的类型的Bean不是唯一的,则会根据属性名寻址Bean名，而后注入。
```

##### 	例子

```
public Interface Animal{
  
}
public Class dog Implements Animal{
  
}
public Class cat Implements Animal{
  
}
@Autowired
private Animal dog;
首先根据类型Animal寻找，有dog,cat两个Bean，则根据属性名和Bean的名称匹配，注入dog。
```

##### 	@Primary和@Quelifier

```
但同一类型（接口）有多个Bean时：
被@Primary注解标注的类具有优先注入功能。
@Quelifier("dog")和@Autowired联合使用，可以明确注入那个Bean.
```

#### 4、Bean的生命周期

```
当DataSource被转配到IOC容器中时，获取到的Bean对象在使用完后需要释放连接，并可以在bean的init和destory方法前后做操作，所以Bean需要有生命周期。
Bean的定义----Bean初始化----Bean的生存期----Bean的销毁
```

#### 5、属性文件

##### 	@ConfigurationProperties(prefix = "")

```
被@ConfigurationProperties注解标注的类能够将属性文件中的配置一一映射到当前类的相应的属性。
默认从全局配置文件中获取值
当前类必须是IOC容器中的组件（Component），才能享受IOC容器提供的@ConfigurationProperties的功能。

<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
```

##### 	@PropertySource（value = {"classpath:xxx.properties"})

```
加载指定的配置文件,可指定多个配置文件。
```

##### 	@ImportResource

```
加载Spring配置文件（被@Configuration组件标注的类）
```

#### 6、条件转配Bean

##### 	@Conditional（xxx.class)

```
被@Condition阿里注解标注的方法在满足条件时才会转配Bean,否则不会转配Bean。
@Conditional注解需要和Condition接口配合使用。
条件类必须实现Condition接口。
```

##### 	例子

```java
public class DatabaseConditional implements Condition{
  @Override
  public boolean matchs(ConditionContext context,AnnotatedTypeMetadata metadata){
    //取出环境配置
    Environment env = context.getEnviroment();
    //判断当前环境下是否有对应数据库的配置
    return env.containsProperty("database.driverName)
    	&& env.contaionsProperty("database.url)
    	&& env.contaionsProperty("database.username")
    	&& env.containsProperty("database.password);
  }
}
@Bean(name = "dataSource", destoryMethod = "close")
@Conditional(DatabaseConditional.class)  //当DatabaseConditional类的matchs方法返回值为true时，IOC容器才会转配这个Bean。
public DataSource getDataSource(
					@value("${database.driverName}") String driver,
					@value("${database.url}") String url),
					...
					){
                      ...
                      return dataSource;
					}
```

#### 7、Bean的作用域

```
常见的有四中作用域：
singleton、prototype、session、application

@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Scope(WebApplicationContext.SCOPE_REQUEST)
```

#### 8、使用@Profile

```
@Profile可以指定项目运行时的环境（开发环境，测试环境，准生产环境，生产环境）
@Profile("dev")或@Profile("test")

Spring中有两个参数能够修改启动Profile机制：
	spring.profiles.active > spring.profiles.default (优先级)
被@Profile注解标注的Bean会在指定的环境下转配到IOC容器中，否则不会装配。

使用yml文件能够以多文档的方式配置相应的开发环境。
server:
  port: 8081
spring:
  profiles:
    active: prod

---
server:
  port: 8083
spring:
  profiles: dev


---

server:
  port: 8084
spring:
  profiles: prod  #指定属于哪个环境
```

#### 9、引入XML配置Bean

```
@ImportResource(value = {"classpath:*.xml"})
```

#### 10、SpringEL

```
SpringEL中，${...}代表占位符，#{...}代表使用Spring表达式。

读取属性文件的值：
	@Value("${database.dreiverName}")
	String driver;
	
	@value("#{T(System).currentTimeMillis()}")
	private long initTime;
	T(...)代表引入类，全类名。
	

```

## Spring AOP约定编程

#### 1、AOP的概念

```
AOP--面向切面编程，在一个方法执行的前后，异常结束做出一些动作。

例如数据库的事务管理：
获取数据库连接对象（前）、异常（回滚）、提交（后）、关闭连接（结束）
使用AOP能够省略try..catch..finally语句。
```

**Spring AOP是一种基于方法的AOP，它只能应用在方法上。**

#### 2、AOP术语和流程

```
术语：
	连接点（join point):具体被拦截（AOP的对象（方法）
	切点（point cut)：在连接点（方法）的什么地方执行AOP（通知）
	通知（advice):前置通知（before advice)、后置通知（after advice)、环绕通知(around advice)、返回通知（afterReturning advice)、异常通知（afterThrowing advice）
	目标对象（target):代理对象
	切面（aspect）:各个通知的集合
```

#### 3、@Aspect

```
Spring以注解@Aspect作为切面声明，当一个类被@Aspect注解标注时，Spring就会知道当前类是一个切面，我们就可以使用各种通知了。
```

#### 4、定义切点

```
@Pointcut("execution([权限修饰符] [返回值类型] [简单类名/全类名] [方法名]([参数列表]))")

例子：
@Pointcut("execution(*com.springboot.chapter.UserServiceImpl.*(..))")
//代表任意的权限和返回值，UserServiceImpl类下的任意方法,".."表示匹配任意多个参数。
public void pointCut(){
	//切点是一个空方法，能够使用方法名被引用此切入点表达式
}
```

#### 5、定义多个切点

```
@Order(1)
public class Aspect1{
  
}
使用@Order注解可以指定切点的执行顺序。
```

# Spring-boot-starter-jpa

### 1、概述

```
Spring Boot中的JPA是依靠Hibernate才能够实现的。
JPA维护的核心是实体(Entity Bean)，实体直接操作API就能够实现对实体对象的CRUD操作，进而完成对象的持久化和查询（面向对象查询语言JPQL)
```

```java
//标明是一个实体类
@Entity(name="user")
//定义映射的表
@Table(name="t_user")
public class User{
  //标明主键
  @Id
  //主键策略，递增
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private long id = null;
  //定义属性和表的映射关系
  @Column(name = "user_name")
  private String userName = null;
  //当属性名与表字段相同时，会一一映射。
  private String note = null;
  //定义转换器
  @Convert(converter = SexConverter.class)
  private SexEnum sex = null;
}
```

**JPA最顶级的接口是Repository(没有定义任何方法，其子接口定义方法），我们定义JPA接口继承JpaRepository接口就可以获得JPA提供的方法了**

```
实现JapRepository接口的JPA接口不需要任何的实现类，因为系统默认帮我们实现了方法，可以直接调用。
//启用JPA编程，将实现JpaRepository接口的接口扫描并转配到IOC容器中，才能进行注入。
@EnableJpaRepositorites(basePackages = "com.springboot.dao)
//将实体类注册给JPA
@EntityScan(basePackage = "com.springboot.pojo")

```

### SpringBoot整合Mybatis

##### 1、导入依赖

```xml
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis‐spring‐boot‐starter</artifactId>
  <version>1.3.1</version>
</dependency>
```

##### 2、配置数据源（druid)

```
spring.datasource.type
```

```java
@Mappper
public interface DepartmentMapper{
  @Select("select * from department where id=#{id}")
  public Department getDeptById(Integer id);
}
@Delete、@Update、@Insert()、@Option(useGeneratedKeys = true,keyProperty = "id")
  
//使用MapperSan批量扫描所有的Mapper接口
  @MapperScan(value = "com.springboot.mapper")
```

#### 使用配置文件整合

```yml
mybatis:
	config-location: classpath:mybatis/mybatis-config.xml
	mapper-location: classpath:mybatis/mapper/*.xml
```

# SpringBoot事务管理

### 1、隔离级别

```
数据库事务具有4个基本特征：原子性、一致性、隔离性、持久性
隔离级别：
	未提交读：允许一个事务读取另外一个事务没有提交的数据
	读写提交：一个事务只能读取另外一个事务已经提交的数据，不能读取未提交的数据
	可重复读：一个事务必须等待另外一个事务提交后才允许读取数据
	串行化：要求所有的SQL都会按顺序执行
```

```java
//串行化隔离级别
@Transactional(isolation = Isolation.SERILIZABLE)

SpringBoot可以通过配置文件制定隔离级别

//隔离级别数字配置的含义：
-1 数据库默认的隔离级别（Mysql--可重复读)
1 未提交读
2 读写提交
4 可重复读
8 串行化
//tomcat数据源默认的隔离级别
spring.datasource.tomcat.default-transaction-isolation=2
//DBCP2数据源默认的隔离级别
spring.datasource.dbcp2.default-transaction-isolation=2
```

### 2、传播行为

```
常用的3种传播行为：
REQUIRED：需要事务，子方法沿用当前事务，默认的传播行为
REQUIRES_NEW：无论当前事务是否存在，都会创建新事务运行方法
NESTEN：在当前方法调用子方法时，如果子方法异常，则回滚子方法的事务，不回滚当前方法的事务
```

