# spring security

标签（空格分隔）： spring security

---

spring security 用于登录认证和授权，注意使用的协议。
##使用security默认登录页面
1.导入依赖
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

2.写一个配置类WebSecurityConfig.java 
继承 WebSecurityConfigurerAdapter
```
package com.example;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;


@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

	@Autowired
	public void configuresGlobal(AuthenticationManagerBuilder auth) throws Exception {
		auth.inMemoryAuthentication().withUser("root").password("root").roles("USER")
		.and().withUser("rui").password("666").roles("ADMIN")
		.and().withUser("xiao").password("777").roles("USER");

	}

	protected void configure(HttpSecurity http) throws Exception {
		
		  http.authorizeRequests() 
		  .anyRequest().authenticated() 
		  .and() .formLogin()
		  .and() .httpBasic();
	}
}

```
运行后跳转security自己的的流入页面
##使用自定义登录页面
1.Controller.java
```
@RequestMapping(value="/login",method=RequestMethod.GET)
public String toLogin() {
	return "login";
}

//@RequestMapping("/")
@RequestMapping("/home")
public String login(String username,String password) {
	System.out.println(username+"--"+password);
	return "loginSuccess";
}
```
2.配置类WebSecurityConfig.java 中的configure方法改为
```
protected void configure(HttpSecurity http) throws Exception {

		http.authorizeRequests()
		.antMatchers("/webjars/**", "/signup", "/about").permitAll()
		.anyRequest()
		.authenticated()
		.and().formLogin()
		.loginPage("/login").permitAll()
		.successForwardUrl("/home")//设置登录后跳转的路径，如果没有这个，那么默认的路径就是“/”,在controller中体现
		.and()
		.logout();
		http.csrf().disable();
		//关闭csrf，如果不关闭，那么在表单提交中加上
		// <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" th:if="${_csrf}" />

	}
```
3.login.html
form表单的action 和method，以及姓名和密码的name属性都是默认的，固
定的。
```
<form action="/login" method="post">
    姓名：<input type="text" name="username" value=""/>
    密码：<input type="password" name="password" value=""/>
    <input type="submit"  value="提交"/>
</form>
```
##使用美化的登录页面
1.引入依赖
```
<dependency>
		<groupId>org.webjars</groupId>
		<artifactId>bootstrap</artifactId>
		<version>3.2.0</version>
</dependency>
```
2.login.html
```
<!DOCTYPE html >
<html xmlns:th="http://www.thymeleaf.org">
<head>
<title>Login</title>
<meta charset="UTF-8"></meta>
<link rel="stylesheet" th:href="@{/webjars/bootstrap/3.2.0/css/bootstrap.min.css}" 
href="/webjars/bootstrap/3.2.0/css/bootstrap.min.css" />
</head>
<style>
<!--
body {
	padding-top: 20px;
}
-->
</style>
<body onload="document.form.username.focus();">
	<nav class="navbar navbar-default" role="navigation">
		<div class="container-fluid">
			<div class="navbar-header">
				<a class="navbar-brand" href=" ">**系统登录</a>
			</div>
		</div>
	</nav>
	<div class="container">
		<div class="row">
			<div class="col-md-4 col-md-offset-4">
				<div class="panel panel-default">
					<div class="panel-heading">
						<h3 class="panel-title">Please sign in</h3>
					</div>
					<div class="panel-body">
						<form name="form" action="login" method="POST">
							<fieldset>
								<div class="form-group">
									<input class="form-control" placeholder="username"
										name="username" type="text" />
								</div>
								<div class="form-group">
									<input class="form-control" placeholder="Password"
										name="password" type="password" value="" />
								</div>
								<div class="checkbox">
									<label> <input name="remember" type="checkbox"
										value="Remember Me" /> Remember Me
									</label>
								</div>
								<input class="btn btn-lg btn-success btn-block" type="submit"
									value="Login" /> <input type="hidden"
									th:name="${_csrf.parameterName}" th:value="${_csrf.token}"
									th:if="${_csrf}" />
							</fieldset>
						</form>
						<div th:if="${session.SPRING_SECURITY_LAST_EXCEPTION!=null}">
						<div class="alert alert-danger alert-dismissable">
							<button type="button" class="close" data-dismiss="alert"
								aria-hidden="true">&times;</button>
							<span th:text="${session.SPRING_SECURITY_LAST_EXCEPTION.message}" ></span>
						</div>
						<input type="hidden" th:name="${_csrf.parameterName}"
							th:value="${_csrf.token}" th:if="${_csrf}" />
					</div>
					</div>
				</div>
			</div>
		</div>
	</div>

</body>
</html>
```
3.给静态文件授权,不登录就可以获取
application.properties
```
security.ignored[0]=/css/*
security.ignored[1]=/js/**
security.ignored[2]=/images/*
security.ignored[3]=/fonts/**
security.ignored[4]=/**/favicon.ico
security.ignored[5]=/**/**.png
security.ignored[6]=webjars/**
```
4.loginSuccess.html
显示当前用户的姓名以及所拥有的权限
权限一般写法（以USER为例）：ROLE_USER
```
Logged user: <span sec:authentication="name">Bob</span>
Roles: <span sec:authentication="principal.authorities">[ROLE_USER, ROLE_ADMIN]</span>
```
为了实现上面的值的显示要引入依赖
```
<dependency>
	<groupId>org.thymeleaf.extras</groupId>
	<artifactId>thymeleaf-extras-springsecurity4</artifactId>
</dependency>
```
##表达式控制URL权限
###在HTML页面实现权限控制
例如：
```
<ul sec:authorize="isAnonymous()" class="nav navbar-nav navbar-right">
	<li><a th:href="@{/login}">login</a> </li>
	<li><a th:href="@{/register}">sign up</a> </li>
</ul>
<ul sec:authorize="isAuthenticated()" class="nav navbar-nav navbar-right">
	<li><a th:href="@{/logout}">logout</a></li>
</ul>
<span sec:authorize="hasAnyRole('ROLE_ADMIN')" >tt</span>
<span sec:authorize="hasRole('ROLE_USER')" >oo</span>
```
| 表达式        | 描述    |
| ---   | -----:    |
| hasRole([role])  |  当前用户是否拥有指定角色   |
| hasAnyRole([role1,role2])| 多个角色是一个以逗号进行分隔的字符串。如果当前用户拥有指定角色中的任意一个则返回true。|
| hasAuthority([auth]) | 等同于hasRole  |
| hasAnyAuthority([auth1,auth2])| 	等同于hasAnyRole|
|  permitAll|  	总是返回true，表示允许所有的 |
|   denyAl|  	总是返回false，表示拒绝所有的   |
|    isAnonymous()     | 当前用户是否是一个匿名用户    |
|    isRememberMe()     |  表示当前用户是否是通过Remember-Me自动登录的   |
|   isAuthenticated()      | 表示当前用户是否已经登录认证成功了。    |
|     Principle    | 代表当前用户的principle对象    |
|     authentication    | 直接从SecurityContext获取的当前Authentication对象    |
|     denyAll    | 总是返回false，表示拒绝所有的    |
|isFullyAuthenticated()|如果当前用户既不是一个匿名用户，同时又不是通过Remember-Me自动登录的，则返回true。|

###在配置类里面使用表达式控制权限
###在service中控制方法的权限
```
package com.example.service;

import org.springframework.security.access.annotation.Secured;
import org.springframework.stereotype.Service;

@Service
public class LoginService {

	@Secured("ROLE_USER")
	public void addUser() {
		System.out.println("user home");
	}
	@Secured("ROLE_ADMIN")
	public void addAmdin() {
		System.out.println("ADMIN home");
	}
}

```

注意：在配置类上要使用注解：
@EnableGlobalMethodSecurity(securedEnabled=true) 

在Controller中调用Service的方法时，只有当前登录用户的service方法@Secured中的相匹配，才可以运行该方法。

