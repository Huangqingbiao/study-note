# Spring Security

### 概述

Spring Security是一种基于SpringAOP和Servlet过滤器的安全框架，同时在Web请求级和方法调用级处理身份确认和授权

**Spring Security框架可以被分为以下几块**

```
Web/Http安全：通过建立Filter和相关的service bean来实现框架的认证机制，当访问受保护的URL时会将用户引入登录页面或错误提示页面
业务对象或方法的安全：控制方法访问权限
AccessDecisionManager:为Web或方法的安全提供访问决策。会注册一个默认的，但额我们也可以通过普通的bean注册方式使用自定义的AccessDecisionManager
AuthenticationManager:处理来自于框架其他部分的认证请求
AuthenticationProvide:AuthenticationManager是通过它来认证用户的
UserDetailsService：跟AuthenticationProvider关系密切，用来获取用户信息

```

