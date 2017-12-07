# Spring 国际化和mobile

标签（空格分隔）： 国际化 mobile

---
1.根据电脑默认语言配置实现语言的展现
Application.java
```
@SpringBootApplication
public class TestThymeleafApplication extends WebMvcConfigurerAdapter{
	@Bean(name = "messageSource")
	public ResourceBundleMessageSource getMessageResource() {
		ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
		messageSource.setBasename("messages");
		return messageSource;
	}
```
Controller.java
```
@RequestMapping("/test")
	public String toLang() {
		return "test";
	}
```
application.properties
```
spring.messages.basename=massages
```
test.html
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8"></meta>
<title>Insert title here</title>
</head>
    <body>
        <h2 th:text="#{welcome}"></h2>
        <h2 th:text="#{name}"></h2>
    </body>
</html>
```

messages_zh_CN.properties
```
welcome=\u6B22\u8FCE\u4F60\u6765\u4E2D\u56FD
name=\u5C0F\u590F
```
messages_en_US.properties
```
welcome=welcome to china
name=xiaxi
```
注意：messages_zh_CN.properties 和messages_en_US.properties是默认固定格式

2.修改配置文件设置语言
application.properties
```
lang=2
```
Application.java
```
@SpringBootApplication
public class TestThymeleafApplication extends WebMvcConfigurerAdapter{

	@Value("${lang}")
	private String lang;
	  @Bean
	  public LocaleResolver localeResolver() {
		  FixedLocaleResolver slr = new FixedLocaleResolver();
		  if(lang.equals("1")) {
			  slr.setDefaultLocale(Locale.CHINA);
		  }else if(lang.equals("2")) {
			  slr.setDefaultLocale(Locale.US);
		  }
		return slr;
	  }
	}
```
messages_zh_CN.properties 和messages_en_US.properties配置文件同上
Controller.java和test.html同1上
修改lang的值实现语言切换

3.点击按钮实现语言切换
Application.java
```
@SpringBootApplication
public class TestThymeleafApplication extends WebMvcConfigurerAdapter{
	@Bean
	public LocaleResolver localeResolver() {
		SessionLocaleResolver  slr = new SessionLocaleResolver();
		slr.setDefaultLocale(Locale.CHINA);
		return slr;

	}
}
```
Controller.java
```
package com.example.controller;

import java.util.Locale;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.i18n.SessionLocaleResolver;

@Controller
public class MessageController {

	@Autowired
	private MessageSource messageSource;
	
	@RequestMapping("/messageTest")
	public String messageTest(String lang,Model model,@RequestHeader("accept-language") String alg) {
		return "message";
	}
	
	
	@RequestMapping("/changeSessionLanguage")
	public String changeSessionLanguage(HttpServletRequest request,String lang) {
		if("zh".equals(lang)) {
			request.getSession().setAttribute(SessionLocaleResolver.LOCALE_SESSION_ATTRIBUTE_NAME, new Locale("zh","CN"));
		}else if("en".equals(lang)) {
			request.getSession().setAttribute(SessionLocaleResolver.LOCALE_SESSION_ATTRIBUTE_NAME, new Locale("en","US"));
		}
		
		
		return "message";
	}
	
}
```

message.html
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8"></meta>
<title>Insert title here</title>
</head>
<body>
 <a href="/changeSessionLanguage?lang=en">English</a>
 <a href="/changeSessionLanguage?lang=zh">简体中文</a>
<br/>
<h2 th:text="#{welcome}"></h2>
<h2 th:text="#{name}"></h2>
</body>
</html>
```
messages_zh_CN.properties 和messages_en_US.properties配置文件同上

---------------------
# spring mobile

---
Controller.java
```
@Controller
public class MobileController {

	@RequestMapping("/pc")
	public String toMobile() {
		return "testMobile";
	}
}
```

Application.java
```
@Autowired
ThymeleafViewResolver  thymeleafViewResolver;
@Autowired
Environment environment;
	
@Bean
public LiteDeviceDelegatingViewResolver liteViewResolver() {
		LiteDeviceDelegatingViewResolver resolver = new LiteDeviceDelegatingViewResolver(thymeleafViewResolver);
		RelaxedPropertyResolver env = new RelaxedPropertyResolver(environment,"spring.mobile.devicedelegatingviewresolver");
		
		
		resolver.setNormalPrefix(env.getProperty("normal-prefix",""));
		resolver.setNormalSuffix(env.getProperty("normal-suffix",""));
		resolver.setMobilePrefix(env.getProperty("mobile-prefix","mobile/"));
		resolver.setMobileSuffix(env.getProperty("mobile-suffix",""));
		/*resolver.setTablePrefix(env.getProperty("table-prefix","table/"));
		resolver.setTableSuffix(env.getProperty("table-suffix",""));*/
		resolver.setEnableFallback(true);
		return resolver;
	}
	
@Bean
public ViewResolver contentNegotiatingViewResolver(ContentNegotiationManager manage) {

		ArrayList<ViewResolver> resolvers = new ArrayList();
		resolvers.add(liteViewResolver());
		ContentNegotiatingViewResolver  resolver = new ContentNegotiatingViewResolver();
		resolver.setViewResolvers(resolvers);
		resolver.setContentNegotiationManager(manage);
		List<View> views = new ArrayList<View>();
		//views.add(MappingJackson2JsonView());
		return resolver;
		
	}

```
两个名字相同的HTML页面testMobile.html
mobile 端的默认放在templates/modile文件夹下；pc端还是放在templates下。

mobile的testMobile.html
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8"></meta>
<title>Insert title here</title>
</head>
    <body>
        mobile页面
    </body>
</html>
```
pc的testMobile.html
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8"></meta>
<title>Insert title here</title>
</head>
    <body>
        pc页面
    </body>
</html>
```
使用手机和pc会跳转不同的页面