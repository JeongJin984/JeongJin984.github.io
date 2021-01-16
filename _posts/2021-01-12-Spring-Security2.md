---
title: Spring Security2
author: Jiny
date: 2021-01-09 13:09:00 +0800
categories: [Java, Spring]
tags: [java, spring, security]
toc: false
---

# Spring Security-2
___

## DelegatingFilterProxy

![image](https://img1.daumcdn.net/thumb/R800x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc4Ruyp%2FbtqIBZqOTEb%2FQ7KNDDJk5rKakxgbAXoRE1%2Fimg.png)

> Spring Security가 모든 App 요청을 감싸게 해서 보안이 적용되게 하는 서블릿 필터

- 서블릿 필터는 스프링에서 정의된 빈을 주입해서 사용할 수 없음
- 특정한 이름을 가진 스프링 빈을 찾아 그 빈에게 요청을 위임
    - springSecurityFilterChain 이름으로 생성된 빈을 ApplicationContext에서 찾아 요청을 위임
    - 실제 보안처리는 하지 않음

```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

이런식으로 필터를 적용하여 모든 요청을 Spring Security가 감싸서 처리할 수 있다.
- DelegatingFilterProxy는 filtername으로 명시된 Spring-Context에서 빈으로 등록된 SpringFilterChain에게 실제 인증처리를 위임하기만 함
- springSecurityFilerChain은 Filter 인터페이스를 구현한 여러종류의 Filter들을 Filer-List 객체에 가지고 있음

|필터|하는 일|
|------|---|
|SecurityContextPersistenceFilter|SecurityContextRepository에서 SecurityContext를 로드하고 저장하는 <br/> 일을 담당함|
|LogoutFilter|로그아웃 URL로 지정된 가상URL에 대한 요청을 감시하고 매칭되는 <br/> 요청이 있으면 사용자를 로그아웃시킴|
|UsernamePasswordAuthenticationFilter|사용자명과 비밀번호로 이뤄진 폼기반 인증에 사용하는 가상 URL <br/>요청을 감시하고 요청이 있으면 사용자의 인증을 진행함 |
|DefaultLoginPageGeneratingFilter|폼기반 또는 OpenID 기반 인증에 사용하는 가상URL에 대한 요청을 <br/>감시하고 로그인 폼 기능을 수행하는데 필요한 HTML을 생성함 |
|BasicAuthenticationFilter|HTTP 기본 인증 헤더를 감시하고 이를 처리함|
|RequestCacheAwareFilter|로그인 성공 이후 인증 요청에 의해 가로채어진 사용자의 원래 <br/>요청을 재구성하는데 사용됨 HttpServletRequest를<br/> HttpServletRequestWrapper를 상속하는 하위 클래스<br/>(SecurityContextHolderAwareRequestWrapper)로 감싸서 필터 <br/>체인상 하단에 위치한 요청 프로세서에 추가 컨텍스트를 제공함  |
|AnonymousAuthenticationFilter|이 필터가 호출되는 시점까지 사용자가 아직 인증을 받지 못했다면<br/> 요청 관련 인증 토큰에서 사용자가 익명 사용자로 나타나게 됨 |
|SessionManagementFilter|인증된 주체를 바탕으로 세션 트래킹을 처리해 단일 주체와 관련한<br/> 모든 세션들이 트래킹되도록 도움|
|ExceptionTranslationFilter|이 필터는 보호된 요청을 처리하는 동안 발생할 수 있는 기대한<br/> 예외의 기본 라우팅과 위임을 처리함|
|FilterSecurityInterceptor|이 필터는 권한부여와 관련한 결정을 AccessDecisionManager에게<br/> 위임해 권한부여 결정 및 접근 제어 결정을 쉽게 만들어 줌|

## Filter 초기화 와 다중 설정 클래스

![image](https://docs.spring.io/spring-security/site/docs/current/reference/html5/images/servlet/architecture/multi-securityfilterchain.png)

- 각각의 Config 파일 별로 다른 Security Filter가 생성되고 이 Filer는 antMatch 메서드(Request Matcher 설정)로 각각 별도의 URL에 매핑됨
- 모든 필터들은 SecurityFilters라는 리스트에 담기게 되고 이는 FilterChainProxy가 가지고 있음
    - 즉 URL을 FilterChainProxy가 받아서 리스트에서 어떤 SecurityFilter를 사용할지를 결정

## Filter Process

1. WebSecurity Class: FilterChainProxy를 Bean으로 등록(new FilterChainProxy(securityChains))
    - securityChains 안에 필터와 requestMatcher가 있음
2. 초기화가 끝나고 Request가 들어오면 FilterProxy에서 사용할 Filter를 고름(getFilters())

## Authentication

> 누구인지를 증명하는 것

- 사용자의 인증 정보를 저장하는 토큰 개념
- 인증 시 id 와 password를 담고 인증 검증을 위해 전달되어 사용
- 인증 후 최종 인증 결과(user 객체, 권한 정보)를 담고 Security Context에 저장되어 전역적으로 참조가 가능
    - Authentication authentication = SecurityContextHolder.getContext().getAuthentication()
- 구조
    - principal: 사용자 id 혹은 User 객체 저장
    - credential: 사용자 비밀번호
    - authorities: 인증된 사용자의 권한 목록
    - details: 인증 부가 정보
    - Authenticated: 인증 여부 

## Authentication Process

![image](https://miro.medium.com/max/1250/1*oVUa7n_m5LkLRzcE-jN_EQ.png)

1. login 정보(id, password)가 전달 됨
2. Filter가 Request를 가로채어 Authentication이라는 객체를 생성
    - FormLogIn이면 UsernamePasswordAuthenticationToken이 생성됨
    - Manager에 전달
3. AuthenticationManager(I)가 ProviderManager(C)를 통해 AuthenticationProvider 목록 중에서 인증 처리 요건에 맞는 AuthenticationProvider를 찾아 인증처리를 위임
    - 부모 ProviderManager를 설정하여 AuthenticationProvider를 계속 탐색 할 수 있음.
    - OAuth일 경우 parent 속성을 통해 부모격의 Provider에서 OAuth 인증 Provider를 사용
        - 초기화 과정에서 2개의 ProviderManager를 만드는데 하나가 부모용 하나가 Default
        - Provider 찾다가 Default에 없으면 부모로 가서 찾음
        - Token의 타입을 통해 Provider를 찾음
    - AuthenticationManagerBuilder를 통해 생성됨(Provider가 생성됨)
4. AuthenticationProvider(I)를 상속한 무슨무슨AuthenticationProvider(C)가 인증처리를 수행
    - provider.authenticate(authentication)
        - ID 검증: UserDetailsService에서 수행(UserDetails 반환)
        - password 검증: 실패 시 BadCredentialException
        - 추가 검증
    - supports: 인증을 처리할 수 있는 Provider인지 확인
4. UserDetailsService에서 실제 인증 처리 역할
    - 유저 유효성 체크
5. Repository에서 유저 객체 조회, UserDetails 타입으로  반환
6. UserDetailsService의 UserDetails로 Provider가 Authentication 객체를 만듬(권한 목록이 채워지는등의 정보가 달라짐)
    - 최종적으로 필터에게 전달
7. SecurityContextHolder의 SecurityContext의 Authentication 객체에 저장됨(필터에서 처리함)

## SecurityContext, Security Context Holder

> SecurityContext: Authentication 객체가 저장되는 Context

- ThreadLocal에 저장되어 아무곳에서나 참조가 가능하도록 설계
    - ThreadLocal: 각 Thread에 할당되는 메모리(공유 불가)
- 인증이 완료되면 HttpSession에 저장되어 App 전반에 걸쳐 전역적인 참조가 가능

- Security 객체 저장 방식(SecurityContextHolder)
    - MODE_THREADLOCAL: thread당 SecurityContext 객체를 할당, 기본값
    - MODE_INHERITABLETHREADLOCAL: 메인 Thread 와 자식 Thread에 관하여 동일한 SecurityContext 유지
    - MODE_GLOBAL: 응용 프로그램에서 단 하나의 SecurityContext를 저장
- SecurityContextHolder.cleanContext(): SecurityContext 기존 정보 초기화

## Security Process

1. Login 요청이 들어오면 Server가 Thread를 생성
    - ThreadLocal이라는 저장소 생성
2. 인증처리(Filter): Authentication 객체 생성
3. 인증 성공 시 SecurityContext에 Authentication 객체 저장
4. HttpSession에 저장
    - Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    - SecurityContext sc = (SecurityContext)session.getAttribute(HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY);
    - Authentication authentication = context.getAuthentication();
    - 2가지 방법으로 authentication 객체 획득 가능

## SecurityContextPersistenceFilter

**SecurityContext 객체의 생성, 저장, 조회**

- 익명 사용자
    - 새로운 SecurityContext를 생성하여 SecurityContextHolder에 저장(SecurityContextPersistenceFilter가 수행)
    - AnonymousAuthenticationFilter에서 AnonymousAuthentificationToken를 SecurityContext에 저장
- 인증 시
    - 새로운 SecurityContext를 생성하여 SecurityContextHolder에 저장(SecurityContextPersistenceFilter가 수행)
    - UsernamePasswordAuthentificationFilter에서 인증 성공 후 해당 토큰 SecurityContext에 저장
    - 인증이 최종 완료되면 Session에 SecurityContext 저장
- 인증 후
    - Session에서 SecurityContext 꺼내서 SecurityContextHolder에 저장(SecurityContextPersistenceFilter가 수행)
    - SecurityContext 안에 Authentication 객체가 존재하면 계속 인증을 유지
- 최종 응답 시 공통
    - SecurityContextHolder.clearContext();

## SecurityContextPersistenceFilter Process

1. FilterChain안의 SCPF가 요청을 받음
2. HttpSecurityContextRepository가 요청을 전달 받음
3. 인증 상태 확인
4. 새로운 SecurityContext 생성
5. chain.doFilter를 통해 인증 필터(AuthFilter)를 통과하게 됨(인증 작업)
6. 인증 후 SecurityContext에 Authentication 객체를 저장
7. Session에 SecurityContext 저장
8. SecurityContext를 holder에서 삭제
9. 응답

## Authorization

> 무엇이 허가 되었는가

- Spring Security가 지원하는 권한 계층
    - 웹계층: URL 요청에 따른 메뉴 혹은 화면 단위의 레벨 보안
    - 서비스 계층: 화면 단위가 아닌 메서드 같은 단위의 레벨 보안
    - 도메인 계층: 객체 단위의 레벨 보안


- FilterSecurityInterceptor
    - 마지막에 위치한 필터로서 인증된 사용자에 대하여 특정 요청의 승인/거부 여부를 최종적으로 결정
    - 인증객체 없이 보호자원에 접근을 시도할 경우 AuthenticationException을 발생
    - 인증 후 자원에 접근 가능한 권한이 존재하지 않을 경우 AccessDeniedException을 발생
    - 권한 제어 방식 중 HTTP 자원의 보안을 처리하는 필터
    - 권한 처리를 AccessDesicionManager에 위임


## Authorization Process

1. 요청이 들어오면 FilterSecurityInterceptor가 인증 여부를 체크
2. SecurityMetaSource(C): 사용자가 요청한 자원에 필요한 권한 정보를 조회해서 전달
3. 권한 정보가 존재할 시 AccessDecisionManager(C)로 전달
    - AccessDecisionVoter를 통해 심의 수행
    - 최종 승인 or 거부

## AccessDecisionManager

- 인증정보, 요청정보, 권한정보를 통해 사용자의 자원접근을 허용할 것인지 거부할 것인지 최종 결정하는 주체
- 여러개의 Voter들로 부터 접근허용, 거부, 보류에 해당하는 각각의 값을 리턴받고 판단 및 결정
- 최종 접근 거부 시 예외 발생

**접근 결정의 세가지 유형**

- AffirmativeBased
    - 여러개의 Voter 클래스 중 하나라도 접근 허가로 결론을 내면 허가로 판단한다.
- ConsensusBased
    - 다수표(승인 및 거부)에 의해 최종 결정을 판단한다
    - 동수일 경우 기본은 접근허기이나 allowIfEqualGrantedDeniedDecisions을 false로 설정할 경우 접근 거부로 결정됨
- UnanimouseBased
    - 모든 Voter가 만장일치로 접근을 승인해야 하며 그렇지 않을 경우 접근을 거부

**AccessDecisionVoter**

- Voter가 권한 부여 과정에서 판단하는 자료
    - Authentication: 인증 정보(user)
    - FilterInvocation: 요청 정보(antMatcher("/user"))
    - ConfigAttributes: 권한 정보(hasRole("USER"))
- 결정 방식
    - ACCESS_GRANTED: 접근 허용(1)
    - ACCESS_DENIED: 접근 거부(0)
    - ACCESS_ABSTAIN: 접근 보류(-1)
        - Voter가 해당 타입의 요청에 대해 결정을 내릴 수 없는 경우

## AccessDecisionManager Process

1. FilterSecurityInterceptor(C)가 AccessDecisionManager(I)에 인가 처리 위임
    - decide(authentication, object, configAttributes)
    - 순서대로 Authentication(인증정보), FilterInvocation(요청정보), ConfigAttributes(권한정보)
2. AccessDecisionManager(I)가 AccessDevisionvoter(I)를 통해 위의 3가지 정보를 이용하여 접근 결정
    - AccessDecisionManager(I) 구현체: AffirmativeBased, ConsensusBased, UnanimousBased
    - decide()를 통해 결정
    - voter(I)의 구현체: RoleVoter: supports()를 통해 return


