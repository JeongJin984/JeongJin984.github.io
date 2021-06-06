---
title: Spring-Web-MVC-2
author: Jiny
date: 2020-11-20 00:14:00 +0800
categories: [Java, Spring]
tags: [web, springmvc]
toc: false
---

# **Web MVC2**
---

### DispatcherServlet

> Servlet Container에서 Http를 통해 들어오는 모든 요청을 Presentation 계층의 제일 앞에 둬서 중앙 집중식으로 처리 해 주는 Front Controller

> 즉, Client로부터 어떠한 요청이 오면 Tomcat(톰캣)과 같은 ServletContainer가 요청을 받는데 이때 제일 앞에서 서버로 들어오는 모든 요청을 처리하는 Front Controller

### Why?
- DispatcherServlet은 web.xml의 역할을 축소
  - 기존에는 모든 서블릿에 대해 URL 매핑을 활용하기 위해서 web.xml에 모두 등록해줘야 했으나 DispatcherServlet이 해당 app으로 들어오는 모든 요청을 핸들링

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FOqO1H%2FbtqAWa7sdbC%2Fd2vlhI8l7fAWeIk29mN4CK%2Fimg.png)

### 구조

<pre>
  DispatcherServlet
  |
  |- Servlet WebApplicationContext extends Root WebApplicationContext
  |                
  |- Root WebApplicationContext
</pre>
---
Servlet WAC
- 특정 servlet 스코프 안에서 사용한 IOC Bean Factory
- Root를 상속하기 때문에 Servlet WAC에서 부모 bean을 사용할 수 있다

Root WAC
- 모든 Scope 에서 사용할 IOC BEAN Factory
- 이건 ContextLoaderListener가 만들어줌
---
**example**

Appconfig.java
- Controller를 Bean으로 등록하지 않음(ComponentScan에서 제외)
- 이걸 가지고 자식 Application Context를 만듬

Webconfig.java
- Controller만 Bean으로 등록
- 이걸 가지고 자식 Application Context를 만듬

Root Context의 context-param의 contextConfigLocation의 param-value를 Appconfig로

servlet app을 app이라는 query에 매핑하고
servlet app의 init-param의 contextConfigLocation의 param-value를 Webconfig로

> 이러면 Root Application Context는 Controller 빼고 다 Bean으로 등록되어 있고 자식놈은 Controller만 Bean으로 등록되어 있음

> 이러면 스코프를 지정하는 다양한 Config파일을 통해 Servlet을 만들게 되고 그러면 정확한 Bean Scope 지정이 가능할 것이다.

그런데 보통은 상속구조를 만들어서 사용하지 않음 SprintBoot도 메인에 ComponentScan달아서 그냥 Root로 쓰는 듯?

### DispatcherServlet의 동작

**DispatcherServlet의 초기화**
- 다음의 특별한 타입의 Bean들을 찾거나, 기본 전력에 해당하는 Bean들을 등록한다.
- HandlerMapping: 핸들러를 찾아주는 인터페이스
- HandlerAdapter: 핸들러를 실행하는 인터페이스


**과정**
1. 요청을 분석한다.(locale, theme,multipart)
2. (핸들러 맵핑에게 위임하여) 요청을 처리할 핸들러를 찾는다.
4. 찾아낸 "핸들러 어댑터"를 사용하여 핸들러의 응답을 처리한다.
  - 핸들러의 리턴값을 보고 어떻게 처리할지 판단한다.
    - 뷰 이름에 해당하는 뷰를 찾아서 모델 데이터를 랜더링한다.
    - @ResponseEntity가 있다면 Converter를 사용해서 응답 본문을 만들고
    - @ResponseBody가 있으면 mv가 null 임
5. 부가적으로 예외가 발생했다면, 예외 처리 핸들러에 요청 처리를 위임한다.
6. 최종적으로 응답을 보낸다.