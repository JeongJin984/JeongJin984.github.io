---
title: Spring Web MVC 3
author: Jiny
date: 2021-07-19 16:30:00 +0800
categories: [Java, Spring]
tags: [web, springmvc]
toc: false
---
 
# Spring Web MVC 3

___

## 💿 **Application Context 과 Servlet Context**

### **Application Context**

- Web Application 최상단에 위치하고 있는 Context
- Spring에서 ApplicationContext란 BeanFactory를 상속받고 있는 Context
- Spring에서 root-context.xml, applicationContext.xml 파일은 ApplicationContext 생성 시 필요한 설정정보를 담은 파일 (Bean 선언 등..)
- Spring에서 생성되는 Bean에 대한 IoC Container (또는 Bean Container)
특정 Servlet설정과 관계 없는 설정을 한다 (@Service, @Repository, @Configuration, @Component)
- 서로 다른 여러 Servlet에서 공통적으로 공유해서 사용할 수 있는 Bean을 선언한다.
- **Application Context에 정의된 Bean은 Servlet Context에 정의 된 Bean을 사용할 수 없다.**


### **Servlet-Context (servlet-context.xml)**

- Servlet 단위로 생성되는 context
- Spring에서 servlet-context.xml 파일은 DispatcherServlet 생성 시에 필요한 설정 정보를 담은 파일 (Interceptor, Bean생성, ViewResolver등..)
- URL설정이 있는 Bean을 생성 (@Controller, Interceptor)
- Application Context를 자신의 부모 Context로 사용한다.
- Application Context와 Servlet Context에 같은 id로 된 Bean이 등록 되는 경우, Servlet Context에 선언된 Bean을 사용한다.
- Bean 찾는 순서
  - Servlet Context에서 먼저 찾는다.
  - 만약 Servlet Context에서 bean을 못찾는 경우 Application Context에 정의된 bean을 찾는다.
- **Servlet Context에 정의된 Bean은 Application Context의 Bean을 사용할 수 있다.**


### **나누는 기준**

- Application Context
  - 공통 기능을 할 수 있는 Bean 설정(Datasource, Service)
  - 각 Servlet에서 공유할 수 있는 Bean
- Servlet Context
  - Servlet 구성에 필요한 Bean 설정(Controller, Interceptor, MappingHandler등...)

> 즉 Servlet Context는 Application Context의 성질을 가지고 있다. 그러나 AC가 SC의 하위 컨텍스트로 탐색 시 역행은 불가하다. 그리고 둘 다 컨텍스트 이므로 저장되는 것은 결국 java Object임에 틀림없다.

- SC는 AC를 부모로 가지고 getServletContext()가 추가된 Interface인데 SC에서 호출 할 시 해당되는 Servlet으로 변환된다.
  - 이 메서드가 추가됨과 동시에 서블릿 컨텍스트를 이용한 몇가지 빈 생애 주기 스코프(애플리케이션, 리퀘스트, 세션 등)가 추가되기도 합니다.


![image](https://leejongchan.github.io/assets/img/2020-02-09-spring-servlet-IoC-container/dispatcher-servlet.png)
___

## 💿 **web.xml**

> 서블릿 클래스를 등록하는 곳의 이름을 **web application deployment descriptor** 라고 하는데 이 역할을 하는 것이 web.xml

- WAS 구동 tl /WEB-INF 디렉토리의 web.xml을 읽어 웹 어플리케이션의 설정을 구성하기 위해 존재한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
 <!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
      <!-- applicationContext.xml에서 설정한 Bean은 모든 서블릿, 필터에서 공유 -->
      <context-param>  
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/applicationContext.xml</param-value>
      </context-param>
      
      <!-- Creates the Spring Container shared by all Servlets and Filters -->
      <!-- 서블릿과 필터에 공유 할 수 있도록 리스너를 설정 -->
      <listener>
          <listener-class>
              org.springframework.web.context.ContextLoaderListener
          </listener-class>
      </listener>
    
      <!-- Processes application requests -->
      <servlet> 
          <servlet-name>dispatcherServlet</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class> 
          <init-param>
              <param-name>contextConfigLocation</param-name> 
              <param-value>
                  <!-- dispatcherServlet 생성 시, 서블릿 설정파일 로드 -->
                  /WEB-INF/spring/appServlet/servlet-context.xml
              </param-value>
          </init-param>
          <load-on-startup>1</load-on-startup>
      </servlet>
            
    <!-- dispatcherServlet 대한 url-pattern을 정의 
        /로 들어오는 모든 요청은 DispatcherServlet에서 처리 -->
      <servlet-mapping>  
            <servlet-name>dispatcherServlet</servlet-name>
            <url-pattern>/</url-pattern>
      </servlet-mapping>
</web-app>
```

### **정리하자면...**

1. 요청이 들어온다.(Servlet Container 즉 tomcat 같은 WAS에)
2. Servlet Container가 쓰레드 풀을 사용하여 들어온 요청을 알맞은 쓰레드에 배정한다.
   - 여기서 DispatcherServlet의 doService()가 일을 하는데 DispatcherServlet도 하나만 존재하고 doService()만 각 쓰레드에서 작동한다.
3. doService를 통해 실행된 서블릿은 어플리케이션 컨텍스트를 참고하여 일을 처리한다.
   - Application Context도 쓰레드 별로 생성되는 것이 아닌 하나만 있고 각 쓰레드가 이를 참고하는 듯 하다.
   - Application Context가 Bean의 Life-Cycle을 관리하는데 Bean의 스코프에 따라 관리한다.

___

## 💿 **Thread 와 빈**

그렇다면 하나의 컨텍스트에서 관리되는 빈들은 Thread Safe 할까? 만약 쓰레드 별로 컨텍스트가 생성된다면 그러할 것이다. 하지만 하나의 컨텍스트를 여러 쓰레드가 공유하고 있으며 컨텍스트는 **기본적**으로 Singleton을 사용한다.

즉 공유되는 Singleton Bean은 **기본적**으로 Thread-Safe 하지 않다는 결론이 나온다.


**그에 관한 Stack Overflow 답변**
```
Are Spring Beans Thread Safe?
No.

Spring has different bean scopes (e.g. Prototype, Singleton, etc.) but all these scopes enforce is when the bean is created. For example a "prototype" scoped bean will be created each time this bean is "injected", whereas a "singleton" scoped bean will be created once and shared within the application context. There are other scopes but they just define a time span (e.g. a "scope") of when a new instance will be created.

The above has little, if anything to do with being thread safe, since if several threads have access to a bean (no matter the scope), it would only depend on the design of that bean to be or not to be "thread safe".

The reason I said "little, if anything" is because it might depend on the problem you are trying to solve. For example if you are concerned whether 2 or more HTTP requests may create a problem for the same bean, there is a "request" scope that will create a new instance of a bean for each HTTP request, hence you can "think" of a particular bean as being "safe" in the context of multiple HTTP requests. But it is still not truly thread safe by Spring since if several threads use this bean within the same HTTP request, it goes back to a bean design (your design of a bean backing class).
```
_https://stackoverflow.com/questions/15745140/are-spring-objects-thread-safe_

### **정리하자면...**

위의 그림처럼 DispatcherServlet 과 Application Context는 포함관계에 있는 것이 맞다. 다만 함수 실행은 Thread Pool에서 실행된다.