---
title: Spring Update 2.1
author: Jiny
date: 2021-03-22 20:30:00 +0800
categories: [Java, Spring]
tags: [java, spring, update]
toc: false
---

# Spring Update 2.1

___

## 💿 **용어**

- 출시일: 2018년 10월
- 조요 변경 내역
  - Spring Framework 5.1
  - 자바 11 지원
  - 스프링 데이터 JPA, lazy 모드 지원
    - 스프링 App이 좀 더 빠르게 뜨게 하기 위하여 Repository가 필요할 때 생성되도록 lazy 모드 지원
  - 의존성 변경
    - Hibernate 5.3(JPA 2.2)
    - Tomcat 9
    - Junit 5.2
  - 빈 오버라이딩을 기본으로 허용하지 않도록 변경
  - Actuator에 "/Info"와 "/health" 공개하도록 바뀜
  - 프로퍼티 변경
  - 로깅 그룹

___

## 💿 **의존성 변경**

- 스프링 Framework 5.0 -> 5.1
  - 로거 설정 spring-jcl
  - 컴포넌트 스캐닝 성능 개선 가능한 컴포넌트 인덱스 기능 제공
  - 함수형 프로그래밍 스타일 지원
  - 코틀린 지원
  - 리액티브 프로그래밍 모델 지원
  - JUnit 5 지원
- JUnit 4.12 -> 5.2
  - Jupiter
  - Extension 모델
  - 람다 지원 
- Tomcat 8.5.39 -> Tomcat 9
  - BIO 커넥터 사라지고 NIO 기본으로 사용
  - HTTP/2 지원
  - 웹소켓 2.0 지원
  - 서블릿 4.0 / JSP 2.4 지원
- Hiberante 5.2 -> 5.3
  - JPA 2.2 지원
  - Java 8의 Date와 Time API 지원

___

## 💿 **JCL**

- Logging Facade: 얘가 로거를 골라서 씀 Facade(정면)
  - JCL(거의 안씀)
  - SLF4J(컴파일 타임에 로거 결정)
- Logger: 로거 퍼사드를 골르면 퍼사드가 로거를 고르고 로거가 일을 함
  - JUL 
  - Log4j, Log4j 2(이거) 
  - Logback(이거)


- 문제점 및 해결
  - 기존의 이미 다른 로깅 퍼사드나 로거를 사용 중인 프로젝트는?(즉 현재는 JCL + Logback 일때)
    - SLF4J로 통하는 Bridge(다리)를 놓는다.(JCL-over-SLF4J)
    - Log4J에서 SLF4J로 다리를 놓는다.(Log4j-to-SLF4J)
  - SLF4J가 사용할 로거는 어떻게 정하지?
    - 사용할 로거를 정해준다.(Binder)

**구체적 순서(과거)**

1. 필요없는 의존성 exclusion
2. 레거시 코드용 브릿지(보통 JCL-over-SLF4J) 의존성 추가
3. SLF4J 추가
4. SLF4J 바인더
5. 로거

**Spring-jcl 등장!**

> 위의 순서에 따른 복잡함을 제거하고 싶다.

1. 기존 스프링 프레임워크에서 로깅하는 코드(JCL 사용)를 전부 고칠 수는 없다.
2. JCL 사용을 권장하지 않으며 SLF4J를 쓰길 바란다.
3. 복잡한 설정을 강요하고 싶지 않다.

> 그래서 스프링이 만든 JCL용 브릿지!

- JCL-over-SLF4J의 대체제
- 클래스패스에 Log4J 2가 있다면 JCL을 사용한 코드가 Log4j 2를 사용한다.
- 클래스패스에 SLF4J가 있다면 JCL을 사용한 코드가 SLF4J를 사용한다.

> 인터페이스가 JCL과 같은데 구현부를 보면 위와 같음(브릿지겸 바인더 역할) 껍데기만 JCL임

- Log4J 2를 사용할 때는 별다른 브릿지나 바인더가 필요없다.
- SLF4J를 사용할 때에도 JCL을 굳이 exclusion하거나 JCL용 브릿지를 추가할 필요 없다.

 ___

 ## 💿 **빈 오버라이딩 기본 설정 변경**

**빈 등록 과정**

 1. App에 정의한 빈 등록
 2. 자동 설정이 제공하는 빈 등록

이때 1번이 정의한 빈을 2번 과정에서 등록하는 빈이 재정의 할 수도 있는데 스프링 부트 2.1 이전까지는 기본적으로 허용하였지만 2.1 이후로는 허용하지 않는다.

___

## 💿 **자동 설정 지원**

- 태스크 실행
  - @EnableAsync 사용시 자동 설정(TaskExecutionAutoConfiguration) 적용
  - spring.task.execution 프로퍼티로 제공
  - TaskExecutorBuilder 제공
- 태스크 스케줄링
  - @EnableScheduling 사용 시 자동 설정(TaskSchedulingAutoConfiguration) 적용
  - spring.task.execution 프로퍼티 제공
  - TaskSchedulerBuilder 제공
- 스프링 데이터 JDBC
  - spring-boot-starter-data-jdbc 의존성 추가시 지원
- 그 밖에
  - 카프카 스트림 지원
  - JMS ConnectionFactory 캐시 지원
  - 엘라스틱 서치 REST 클라이언트 지원

___

## 💿 **프로퍼티 변경**

- 스프링 데이터 JPA 부트스크랩 모드 지원
  - APP 구동 시간을 줄이기 위해 스프링 데이터 JPA Repository 생성을 지연 시키는 설정
    - spring.data.jpa.repositories.bootstrap-mode=deferred
    - DEFERRED: APP 구동 이후에 Repository 인스턴스를 만들어 주입해 준다.(이전에는 Proxy 객체를 참조한다.)
    - LAZTY: APP 구동 이후에도 만들지 않다가 처음으로 사용할 시점에 만들어 주입해준다.
- JpaProperties에서 하이버네이트 관련 프로퍼티를 HibernateProperties로 분리
- Migration 툴: 프로퍼티 마이그레이션을 하지 않더라도 기존 프로퍼티로 APP 구동이 가능하며 프로퍼티가 어떻게 바뀌었는지 알려주는 툴
  - spring-boot-properties-migrator

___

## 💿 **Data Size**

- org.springframework.util.unit.DataSize
  - 스프링 부트가 아니라 스프링 프레임워크가 5.1부터 지원하는 타입으로 데이터 사이즈 MB, GB등을 표현하는데 사용할 수 있는 타입이다.
  - 지원하는 타입: B, KB, MB, GB, TB
- 스프링 부트는 컨버터를 지원
  - StringToDataSizeConverter
  - NumberToDataSizeConverter

___

## 💿 **로그 그룹**

> 같은 로그 레벨을 적용할 패키지 묶음을 만들 수 있는 기능으로 여러 패키지의 로그 레벨을 손쉽게 변경할 수 있다.

- 로그 그룹 정의
   - logging.group.{그룹 이름}={패키지}, {패키지}, {패키지}
   - logging.level.{그룹 이름}={로그 레벨}
- 스프링 부트가 미리 정의해둔 로그 그룹
  - web = 스프링 웹MVC 관련 패키지 로그 그룹
  - sql = 스프링 JDBC와 하이버네이트 SQL을 묶어둔 로그 그룹

___

## 💿 **Actuator**

- /Info와 /health 엔드포인트가 스프링 시큐리티를 추가하더라도 기본적으로 "공개"하도록 변경됨
  - 2.0 에서는 시큐리티를 추가하면 모든 엔드포인트가 인증을 거쳐야 했음
  - 물론 설정을 통해 컨트롤 할 수 있음
- /Info 엔드포인트에 정보 추가하는 방법
  1. Info 키값에 들어있는 모든 프로퍼티
  2. git.properties에 들어있는 프로퍼티
  3. META-INFO/build-info.properties에 들어있는 프로퍼티