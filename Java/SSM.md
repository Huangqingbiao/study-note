# Spring整合mybatis的配置文件

1、加载外部属性文件（数据库配置文件）

```java
<context:property-placeholder location="classpath:jdbc.properties "/>
```

2、配置数据源

```
<bean id="dataSource" class="con.alibaba.druid.pool.DruidDataSource">
	<property name="username" value="${jdbc.user}" />
	<property name="password" value="${jdbc.password}"/>
		<property name="url" value="${jdbc.url}"/>
		<property name="driverClassName" value="${jdbc.driver}"/>
	</bean>
```

3、配置SqlSessionFactoryBean整合MyBatis(包含分页插件PageHelper)

```
<bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
		<!-- 指定MyBatis全局配置文件位置 -->
		<property name="configLocation" value="classpath:mybatis-config.xml"/>
		
		<!-- 指定Mapper.xml配置文件位置 -->
		<property name="mapperLocations" value="classpath:mybatis/mapper/*Mapper.xml"/>
		
		<!-- 装配数据源 -->
		<property name="dataSource" ref="dataSource"/>
		
		<!-- 配置插件 -->
		<property name="plugins">
			<array>
				<!-- 配置PageHelper插件 -->
				<bean class="com.github.pagehelper.PageHelper">
					<property name="properties">
						<props>
							<!-- 配置数据库方言，告诉PageHelper当前使用的数据库 -->
							<prop key="dialect">mysql</prop>
							
							<!-- 配置页码的合理化修正，在1~总页数之间修正页码 -->
							<prop key="reasonable">true</prop>
						</props>
					</property>
				</bean>
			</array>
		</property>
	</bean>
```

4、配置MapperScannerConfigurer来扫描Mapper接口所在的包，将Mapper接口加入到IOC容器中，使其能够自动注入（Autowired)，方便使用

```
<bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.atguigu.crowd.mapper"/>
	</bean>
```

# Spring配置事物管理

1、配置自动扫描包，主要将Service扫描到IOC容器中

```
<context:component-scan base-package="com.atguigu.crowd.service"/>
```

2、配置事物管理器

```
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<!-- 装配数据源 -->
		<property name="dataSource" ref="dataSource"/>	
	</bean>
```

3、配置事物切面

```
<aop:config>
		<!-- 考虑到后面我们整合SpringSecurity，避免把UserDetailsService加入事务控制，让切入点表达式定位到ServiceImpl -->
		<aop:pointcut expression="execution(* *..*ServiceImpl.*(..))" id="txPointcut"/>
		
		<!-- 将切入点表达式和事务通知关联起来 -->
		<aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
	</aop:config>
```

4、配置事物通知

```
<tx:advice id="txAdvice" transaction-manager="txManager">
	
		<!-- 配置事务属性 -->
		<tx:attributes>
			
			<!-- 查询方法：配置只读属性，让数据库知道这是一个查询操作，能够进行一定优化 -->
			<tx:method name="get*" read-only="true"/>
			<tx:method name="find*" read-only="true"/>
			<tx:method name="query*" read-only="true"/>
			<tx:method name="count*" read-only="true"/>
			
			<!-- 增删改方法：配置事务传播行为、回滚异常 -->
			<!-- 
				propagation属性：
					REQUIRED：默认值，表示当前方法必须工作在事务中，如果当前线程上没有已经开启的事务，则自己开新事务。如果已经有了，那么就使用这个已有的事务。
						顾虑：用别人的事务有可能“被”回滚。
					REQUIRES_NEW：建议使用的值，表示不管当前线程上有没有事务，都要自己开事务，在自己的事务中运行。
						好处：不会受到其他事务回滚的影响。
			 -->
			<!-- 
				rollback-for属性：配置事务方法针对什么样的异常回滚
					默认：运行时异常回滚
					建议：编译时异常和运行时异常都回滚
			 -->
			<tx:method name="save*" propagation="REQUIRES_NEW" rollback-for="java.lang.Exception"/>
			<tx:method name="update*" propagation="REQUIRES_NEW" rollback-for="java.lang.Exception"/>
			<tx:method name="remove*" propagation="REQUIRES_NEW" rollback-for="java.lang.Exception"/>
			<tx:method name="batch*" propagation="REQUIRES_NEW" rollback-for="java.lang.Exception"/>
			
		</tx:attributes>
	
	</tx:advice>
```

# SpringMVC配置文件

1、配置自动扫描的包，将handle（controller)组件加入到IOC容器中

```
<context:component-scan base-package="com.atguigu.crowd.mvc"/>
```

2、配置SpringMVC的注解驱动

```
<mvc:annotation-driven/>
```

3、配置视图解析器

```
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/"/>
		<property name="suffix" value=".jsp"/>
	</bean>
```

4、配置基于XML的异常映射

```
<bean id="simpleMappingExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
		<!-- 配置异常类型和具体视图页面的对应关系 -->
		<property name="exceptionMappings">
			<props>
				<!-- key属性指定异常全类名 -->
				<!-- 标签体中写对应的视图（这个值要拼前后缀得到具体路径） -->
				<prop key="java.lang.Exception">system-error</prop>
				<prop key="com.atguigu.crowd.exception.AccessForbiddenException">admin-login</prop>
			</props>
		</property>
	</bean>
```

5、配置view-controller，直接将请求地址和视图名称关联起来，不必写Handler方法了

```
<mvc:view-controller path="/admin/to/login/page.html" view-name="admin-login"/>
	<mvc:view-controller path="/admin/to/main/page.html" view-name="admin-main"/>
	<mvc:view-controller path="/admin/to/add/page.html" view-name="admin-add"/>
	<mvc:view-controller path="/role/to/page.html" view-name="role-page"/>
	<mvc:view-controller path="/menu/to/page.html" view-name="menu-page"/>
	
```

6、注册拦截器

```
<mvc:interceptors>
		<mvc:interceptor>
			<!-- mvc:mapping配置要拦截的资源 -->
			<!-- /*对应一层路径，比如：/aaa -->
			<!-- /**对应多层路径，比如：/aaa/bbb或/aaa/bbb/ccc或/aaa/bbb/ccc/ddd -->
			<mvc:mapping path="/**"/>
			
			<!-- mvc:exclude-mapping配置不拦截的资源 -->
			<mvc:exclude-mapping path="/admin/to/login/page.html"/>
			<mvc:exclude-mapping path="/admin/do/login.html"/>
			<mvc:exclude-mapping path="/admin/do/logout.html"/>
			
			<!-- 配置拦截器类 -->
			<bean class="com.atguigu.crowd.mvc.interceptor.LoginInterceptor"/>
		</mvc:interceptor>
	</mvc:interceptors>
```

# SpringMVC中的请求方式

前端发送请求，后端处理数据的方式有两种：

普通请求：后端处理后返回页面，会刷新整个页面内容。

Ajax请求：后端处理后返回json数据,Jquery代码使用json数据对页面局部刷新。

**发送Ajax请求**

```
// 准备要发送的数组
var array = [5,8,12];
// 将 JSON 数组转换成 JSON 字符串
// "[5,8,12]"
var arrayStr = JSON.stringify(array);
// 发送 Ajax 请求
$.ajax({
"url":"",
"type":"post",
"data":arrayStr, // JSON 字符串作为请求体
"contentType":"application/json;charset=UTF-8",// 告诉服务器端当前请求的请求体是
JSON 格式
"dataType":"text",
"success":function(response){
console.log(response);
},
"error":function(response){
console.log(response);
}
});
```

**统一Ajax请求返回类型**

```
/**
* 用于统一项目中所有 Ajax 请求的返回值类型
* @author Lenovo
* *
@param <T>
*/
public class ResultEntity<T> {
public static final String SUCCESS = "SUCCESS";
public static final String FAILED = "FAILED";
public static final String NO_MESSAGE = "NO_MESSAGE";
public static final String NO_DATA = "NO_DATA";
private String operationResult;
private String operationMessage;
private T queryData;
/**
* 返回操作结果为成功， 不带数据
* @return
*/
public static <E> ResultEntity<E> successWithoutData() {
return new ResultEntity<E>(SUCCESS, NO_MESSAGE, null);
} /
**
* 返回操作结果为成功， 携带数据
* @param data
* @return
*/
public static <E> ResultEntity<E> successWithData(E data) {
return new ResultEntity<E>(SUCCESS, NO_MESSAGE, data);
} /
**
* 返回操作结果为失败， 不带数据
* @param message
* @return
*/
public static <E> ResultEntity<E> failed(String message) {
return new ResultEntity<E>(FAILED, message, null);
}

public ResultEntity() {
} 
public ResultEntity(String operationResult, String operationMessage, T queryData) {
super();
this.operationResult = operationResult;
this.operationMessage = operationMessage;
this.queryData = queryData;
} @
Override
public String toString() {
return "AjaxResultEntity [operationResult=" + operationResult + ", operationMessage="
+ operationMessage
+ ", queryData=" + queryData + "]";
}
public String getOperationResult() {
return operationResult;
}
public void setOperationResult(String operationResult) {
this.operationResult = operationResult;
}
public String getOperationMessage() {
return operationMessage;
}
public void setOperationMessage(String operationMessage) {
this.operationMessage = operationMessage;
}
public T getQueryData() {
return queryData;
} 
public void setQueryData(T queryData) {
this.queryData = queryData;
}
}
```

# 异常映射

异常映射要先判断当前请求的类型，如果是普通请求，则在页面返回异常信息，如果是Ajax请求，则返回json数据。

### 判断请求类型

判断依据：请求头中的Accept属性值、X-Request-With属性值。

Ajax请求头：

```
Accept:application/json
X-Request-With:XMLHttpRequest
```

判断当前请求是否为Ajax请求

```
public static boolean judgeRequestType(HttpServletRequest request) {
// 1.获取请求消息头信息
String acceptInformation = request.getHeader("Accept");
String xRequestInformation = request.getHeader("X-Requested-With");
// 2.检查并返回
return
(
acceptInformation != null
&&
acceptInformation.length() > 0
&&
acceptInformation.contains("application/json")
)
||
(
xRequestInformation != null
&&
xRequestInformation.length() > 0
&&
xRequestInformation.equals("XMLHttpRequest")
);
}
```

# 异常映射实现方式

1、基于xml

```
<!-- 配置基于 XML 的异常映射 -->
<bean id="simpleMappingExceptionResolver"
class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
<!-- 指定异常类型和逻辑视图名称的对应关系 -->
<property name="exceptionMappings">
<props>
<!-- key 属性指定异常类型（全类名） -->
<!-- 文本标签体中指定异常对应的逻辑视图名称 -->
<prop key="java.lang.NullPointerException">system-error</prop>
</props>
</property>
<!-- 使用 exceptionAttribute 可以修改异常对象存入请求域时使用的属性名 -->
<!-- <property name="exceptionAttribute"></property> -->
</bean>
```

2、基于注解（加入json依赖）

```
/**
* 基于注解的异常处理器类
* @author Lenovo
* *
/
// 表示当前类是一个处理异常的类
@ControllerAdvice
public class CrowdExceptionResolver {
@ExceptionHandler(value = LoginFailedException.class)
public ModelAndView resolveLoginFailedException(
LoginFailedException exception,
HttpServletRequest request,
HttpServletResponse response
) throws IOException {
// 只是指定当前异常对应的页面即可
String viewName = "admin-login";
return commonResolveException(exception, request, response,
viewName);
} /
/ 表示捕获到 Exception 类型的异常对象由当前方法处理
@ExceptionHandler(value = Exception.class)
public ModelAndView resolveException(
Exception exception,
HttpServletRequest request,
HttpServletResponse response
) throws IOException {
// 只是指定当前异常对应的页面即可
String viewName = "system-error";
return commonResolveException(exception, request, response,
viewName);
} /
**
* 核心异常处理方法
* @param exception SpringMVC 捕获到的异常对象
* @param request 为了判断当前请求是“普通请求”还是“Ajax 请求”
需要传入原生 request 对象
* @param response 为了能够将 JSON 字符串作为当前请求的响应数
据返回给浏览器
* @param viewName 指定要前往的视图名称
* @return ModelAndView
* @throws IOException
*/
private ModelAndView commonResolveException(
Exception exception,
HttpServletRequest request,
HttpServletResponse response,
String viewName
) throws IOException {
// 1.判断当前请求是“普通请求”还是“Ajax 请求”
boolean judgeResult = CrowdUtil.judgeRequestType(request);
// 2.如果是 Ajax 请求
if(judgeResult) {
// 3.从当前异常对象中获取异常信息
String message = exception.getMessage();
// 4.创建 ResultEntity
ResultEntity<Object> resultEntity =
ResultEntity.failed(message);
// 5.创建 Gson 对象
Gson gson = new Gson();
// 6.将 resultEntity 转化为 JSON 字符串
String json = gson.toJson(resultEntity);
// 7.把当前 JSON 字符串作为当前请求的响应体数据返回给
浏览器
// ①获取 Writer 对象
PrintWriter writer = response.getWriter();
// ②写入数据
writer.write(json);
// 8.返回 null， 不给 SpringMVC 提供 ModelAndView 对象
// 这样 SpringMVC 就知道不需要框架解析视图来提供响应，
而是程序员自己提供了响应
return null;
} /
/ 9.创建 ModelAndView 对象
ModelAndView modelAndView = new ModelAndView();
// 10.将 Exception 对象存入模型
modelAndView.addObject(CrowdConstant.ATTR_NAME_EXCEPTION,
exception);
// 11.设置目标视图名称
modelAndView.setViewName(viewName);
// 12.返回 ModelAndView 对象
return modelAndView;
}
}
```

