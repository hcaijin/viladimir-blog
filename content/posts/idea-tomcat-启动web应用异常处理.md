---
title: idea tomcat 启动web应用异常处理
tags: 
- java 
- tomcat 
- idea
permalink: idea-tomcat-qi-dong-webying-yong-yi-chang-chu-li
id: 62
updated: '2016-05-11 05:36:39'
date: 2016-05-11 05:28:04
---

```
2016-05-11 16:36:25.799 [RMI TCP Connection(4)-127.0.0.1] ERROR org.springframework.web.context.ContextLoader - Context initialization failed
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'shiroFilter' defined in file [/home/hcj/Work/data/ecerp-saas/Sources/ecerp/out/artifacts/ecerp_web_war_exploded/WEB-INF/classes/spring/applicationContext.xml]: Cannot resolve reference to bean 'securityManager' while setting bean property 'securityManager'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'securityManager' defined in file [/home/hcj/Work/data/ecerp-saas/Sources/ecerp/out/artifacts/ecerp_web_war_exploded/WEB-INF/classes/spring/applicationContext.xml]: Cannot resolve reference to bean 'shiroSubjectFactory' while setting bean property 'subjectFactory'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'shiroSubjectFactory': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: private gy.erp.service.admin.SecurityMonitor gy.erp.shiro.ShiroSubjectFactory.securityMonitor; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'securityMonitor' defined in class path resource [spring-domain.xml]: Cannot resolve reference to bean 'sessionService' while setting bean property 'sessionService'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sessionService': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalArgumentException: Invalid registry store file /data/erp.guanyisoft.com/tomcat/ecerp-web.properties, cause: Failed to create directory /data/erp.guanyisoft.com/tomcat!
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:329) ~[spring-beans-3.2.0.RELEASE.jar:3.2.0.RELEASE]
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveValueIfNecessary(BeanDefinitionValueResolver.java:107) ~[spring-beans-3.2.0.RELEASE.jar:3.2.0.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyPropertyValues(AbstractAutowireCapableBeanFactory.java:1391) ~[spring-beans-3.2.0.RELEASE.jar:3.2.0.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1132) ~[spring-beans-3.2.0.RELEASE.jar:3.2.0.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:522) ~[spring-beans-3.2.0.RELEASE.jar:3.2.0.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:461) ~[spring-beans-3.2.0.RELEASE.jar:3.2.0.RELEASE]
	at org.
```
java报错总要看异常信息，以上主要关键的地方列在这里：
```
;nested exception is java.lang.IllegalArgumentException: Invalid registry store file /data/erp.guanyisoft.com/tomcat/ecerp-web.properties, cause: Failed to create directory /data/erp.guanyisoft.com/tomcat!
```
idea tomcat 在启动web应用的时候会生成一个注册dubbo服务的文件，需要指定生成路径，以前项目都是默认生成在out文件里的吧，最近，不知道什么变动，需要手工在项目配置文件application.properties 指定一下：
```
dubbo.registry.file = /home/hcj/Work/data/ecerp-web.properties
```
