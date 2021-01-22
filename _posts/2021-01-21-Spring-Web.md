---
title: Spring-Web
author: Jiny
date: 2021-01-21 22:17:00 +0800
categories: [Java, Spring]
tags: [web, springmvc]
toc: false
---

# Spring Web
___

## URL

- get, post는 기본
- head: get과 같으나 header 정보만 가져옴
- options
  - 사용할 수 있는 http Method 제공
  - 서버 또는 특정 리소스가 제공하는 기능을 확인
  - 서버 또는 allow 응답 헤더에 사용할 수 있는 http method 목록 제공

## @Controller

> request를 담당하는 annotation RestController는 자동으로 responsebody에 객체를 담아준다.

- @RequestMapping: Responsebody가 있거나 RestController를 사용하지 않으면 반환되는 문자열에 해당하는 View를 return 하고 Class에다가 선언 시 method의 RequestMapping과 value가 합쳐서 경로가 결정된다.
	- value: request 경로 (uri 패턴 & 정규식 지원),(추가 형식)
  - method: handle할 method
  - consumes: 사용할 MediaType 지정
  - produces: 내보낼 MediaType 지정(overiding됨)
  - headrs: headr가 선언된 조건식에 일치 하는가
  - params: 코딩된 uriParam과 일치하는가
  - 유사: GetMapping, PostMapping...
  - 자동 확장자 매핑 불가
- @RequestParams
  - argument에 사용: uri 패턴의 parameter를 가져옴
  - WebRequest라는 매개변수를 통해 가져올 수 도 있음
- @PathVariable
  - argument에 사용: path에 해당하는 값을 가져옴

## Formatter

1. Formatter를 상속하여 Formatter Class로 overriding 함
2. WebConfigurer를 상속한 Config Class에 addFormatter를 통해 등록

## Converter

> Formatter는 String 대상으로만 사용 가능 Converter는 클래스 대상으로도 사용 가능하다.

- 또한 SpringJPA가 자동으로 등록해 주는 domain Class Converter는 id를 Entity에 매핑 시켜 준다.

## Handler Interceptor

- preHandle(request, response, handler)
  - handler를 실행하기 전에 호출됨
  - handler에 대한 정보를 사용할 수 있기 때문에 서블릿 필터에 비해 보다 세밀한 로직 구현
  - 리턴 값으로 계속 다음 interceptor 또는 handler 요청, 응답을 전달할지 응답처리가 	이곳에서 끝난지를 알림
- postHandle(request, response, modelAndView)
  - 핸들러 실행이 끝나고 뷰를 랜더링 하기 전에 호출됨
  - 뷰에 전달할 추가적이거나 여러 handler에 공통적인 모델 정보를 	담는데 사용할 수 있음
  - 비동기적인 요청 처리 시에는 호출되지 않습니다.
- afterCompletion(request, response, handler, ex)
  - 요청 처리가 완전히 끝난 뒤(뷰 렌더링이 끝난 후) 호출 됨
  - preHandler에서 true를 return한 경우에만 호출 됨
  - 비동기적 요청처리에는 호출되지 않음
- 순서
	1. prehandle 1
	2. prehandle 2
	3. 요청처리
	4. posthandle 2
	5. posthandle 1
	6. 뷰렌더링
	7. aftercompletion 2
	8. aftercompletion 1

+ 서블릿 보다 구체적인 처리 가능
+ 서블릿 보다 일반적인 용도의 기능을 구현

### 구현

- HandlerInterceptor implement 해서 사용
- webMvcConfigurer 상속 클래스에 addInterceptor을 통해 등록
  - order함수를 통해 순서 지정
  - addPathPatterns를 통해 특정 경로에만 match

## 리소스 핸들러

- Default Servlet
  - Servlet Container가 기본적으로 제공하는 Servlet으로 정적인 Resource 처리할 때 사용
- Spring MVC Resource Handler 등록
  - 가장 낮은 순위로 등록
    - 다른 Handler Mapping이 "/"이하 요청을 처리 허용
    - 최종적으로 resource Handler가 처리
- 설정
  - 어떤 요청 패턴 지원할 것인가
  - 어디서 리소스를 찾을 것인가
  - 캐쉬
  - Resource Resolver: 요청에 해당하는 Resource를 찾는 전략
    - Caching, Encoding, WebJar
  - ResourceTransformer: 응답으로 보낼 리소스를 수집하는 전략
    - Caching, CSS 링크, HTML5,
- SpringBoot가 기본적인 핸들러 캐싱 제공

## Http Message Converter

> 요청 본문에서 메세지를 읽거나 응답 본문에 메세지를 작성할 때 사용 jackson같은거

## Arguments

- PushBuilder: 요청이 오면 필요한 리소스를 요청하지 않아도 보내는 기술
- HttpMethod: 요청의 method를 가져옴
- HttpSession: 세션 이용
- 많이 있으니까 doc 참고

## @ModelAttributes

- Request로 넘어오는 여러가지 데이터들을 복합타입 객체로 받아오거나 새로 만들 때 사용
- 값을 바인딩 할 수 없을 땐 BindException 발생
  - 직접 Exception 처리할 땐 BindingResult 타입의 argument
- @Valid를 통해 검증 작업 진행 가능
- Session에서 먼저 찾아서 바인딩 하고 queryparam에서 바인딩 하는데 키가 중복될 경우 session에서 못찾으면 에러가 난다.
  
- Validated의 경우 Valid그룹 지정 가능

## @SessionAttributes

> Controller 안에서 공용으로 사용할 객체를 세션에 담아서 공유

- 요청 정보를 Http세션에 저장
  - 직접사용하여 저장
  - 이름을 설정해서 해당하는 모델 자동으로 세션에 저장
  - @ModelAttribute는 세션에 있는 데이터 바인딩
    - 요청을 따로 보낼 필요가 없다(Form이 나눠져 있을 때)
  - 여러 요청에 사용하는 객체를 공유해야 할 때 사용
- SessionStatus를 사용해서 세션 처리 완료 가능
  - 폼 처리 끝나고 세션 비울때

## @SessionAttribute

> Controller 외부에서(Servlet 등등)에서 저장한 세션값을 가져오고 싶을 때  

- argument에 선언해서 사용한다.

## RedirectAttributes

- Model의 경우 모든 값은 Redirect 시 자동으로 queryParameter로 등록 됨
  - SpringBoot는 이 기능을 꺼놨음
- Model 대신에 RedirectAttriburtes 사용하면 됨
- addflashAttributes를 할 경우 자동으로 세션으로 등록후 Redirect 끝날 때 자동으로 세션 초기화 
  - 받을 때 model.asMap().get("키")

## MultipartFile

- MultipartFile 타입의 매개변수로 받음
- MulipartResolver가 없으면 안됨
- max크기등을 설정할 수 있음(application.properties)