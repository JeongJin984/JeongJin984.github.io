---
title: Spring Security
author: Jiny
date: 2021-01-11 13:09:00 +0800
categories: [Java, Spring]
tags: [java, spring, security]
toc: false
---

# Spring Security
___

## Spring Security?

> 스프링 시큐리티는 스프링 기반의 애플리케이션의 보안(인증과 권한, 인가 등)을 담당하는 하위 프레임 워크

**기본용어**

- 접근 주체: 보호된 리소스에 접근하는 대상
- 인증(Authentication): 보호된 리소스에 접근한 대상에 대해 누구인지, App의 작업을 수행해도 되는 주체인지 확인하는 과정을
    - 즉 누구인지
- 인가(Authorize): 해당 리소스에 대해 접근 권한을 가지고 있는지 확인하는 과정
    - 즉 무엇을 할 수 있는지
- 권한: 인가 과정에서 해당 리소스에 대한 최소한의 권한을 가졌는지 확인


## Spring Security 특징과 구조

- 보안과 관련하여 체계적으로 많은 옵션을 제공하여 편리하게 사용할 수 있음
- Filter 기반으로 동작하여 MVC와 분리하여 관리 및 동작
- Annotation을 통한 간단한 설정
- 기본적으로 세션 & 쿠키 기반으로 동작

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FRdJGx%2FbtqD9Ouzlub%2F5At2yq9zCxACpguIwWKHE1%2Fimg.png)

- 인증 관리자(Authentication Manager)와 접근 결정 관리자(Access Decision Manager)를 통해 Resource 접근을 관리
- 인증 관리자는 UsernamePasswordAuthenticationFilter, 접근 결정 관리자는 FilterSecurityIntercepter가 수행

## Security 구조

![image](https://user-images.githubusercontent.com/22395934/68668144-de371900-058a-11ea-913a-cd912f5ab9df.png)

## Servlet Filter

- 서블릿이 호출되기 전에 요청, 응답을 가로챔
    - 점검
    - 헤더 조작

![image](https://media.vlpt.us/images/sa833591/post/7cf6afac-cc25-4de1-ad33-6fc2175d7e50/SecurityFilterChain.JPG)