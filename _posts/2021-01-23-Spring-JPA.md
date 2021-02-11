---
title: Spring JPA
author: Jiny
date: 2021-01-24 17:16:00 +0800
categories: [Java, Spring]
tags: [java, spring, jpa]
toc: false
---

# Spring JPA
___

## 💿 JPA

![image](https://gmlwjd9405.github.io/images/spring-framework/spring-jpa-architecture.png)

> 자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스

**여기서 주목할 점은 바로 인터페이스라는 것이다**. JPA는 특정 기능을 하는 라이브러리가 아니다. JPA는 자바 어플리케이션에서 관계형 데이터베이스를 어떻게 사용해야 하는지를 정의한 방법이다.



**JPA는 단순히 interface이기 때문에 구현이 없다.** JPA를 정의한 javax.persistence 패키지의 대부분은 각종 Annotation 과 interface, enum, Exception으로 이루어져 있다. JPA의 핵심이 되는 EntityManager는 아래와 같이  javax.persistence.EntityManager라는 파일에 interface로 정의되어 있다.

```java
package javax.persistence;

import ...
;

public interface EntityManager {

    public void persist(Object entity);

    public <T> T merge(T entity);

    public void remove(Object entity);

    public <T> T find(Class<T> entityClass, Object primaryKey);

    // More interface methods...
}
```

### JPA의 3요소

1. javax.persistance 패키지로 정의된 API 그 자체
2. JPQL(Java Persistence Query Language)
3. 객체/관계 메타데이터
___

## 💿 Hibernate

> hibernate는 JPA의 구현체

![image](https://suhwan.dev/images/jpa_hibernate_repository/jpa_hibernate_relationship.png)

EntityManagerFactory, EntityManager, EntityTransaction을 Hibernate에서는 각각 SessionFactory, Session, Transaction으로 상속받고 각각 Impl로 구현한 것이다.

**즉, JPA를 사용하기 위해서 반드시 hibernate를 사용할 필요가 없다.**

### Hibernate 구조

![image](https://gmlwjd9405.github.io/images/spring-framework/spring-hibernate-architecture.png)

1. **Configurattion**
	- Session Factory에서 Java Application 및 Database와 함께 작업할 때 사용
	- It represents an entire set of mappings of an application Java Types to an SQL database.
2. **Session Factory**
   - 사용자 어플리케이션에서 Session Factory에 Session 객체를 요청하면 Session Factory는 Configuration 정보를 사용하여 Session Object를 인스턴스화 한다. 
3. **Session**
   - 응용 프로그램과 데이터베이스 간의 상호작용을 나타낸다.(항시적으로)세션의 인스턴스는 SessionFactory Bean으로 부터 발생
   - Session 객체는 application 과  데이터베이스(데이터가 저장 된)의 인터페이스이다.짧은 생존주기를 가지고 있으며, JDBC Connection 을 포장 하고 있다. (쿼리를 직접적으로 수행하는 것 같다. 그래서 생존주기(트랜잭션??)가 짧으며, JDBC Connection 을 포장한다는 표현을 한 것 같다.)
   - Session, EntityManager 같은 것들이 Persistent Context라고 부른다. 그릭
4. **Query**
   - 응용프로그램이 하나 이상의 저장된 객체(Persistence Object)에 대해 데이터베이스를 쿼리할 수 있게한다.
5. **First-level cache**
   - 데이터베이스와 상호 작용하는 동안 Hibernate Session 객체가 사용하는 기본 캐시를 나타낸다. Session cache라고도 하며, 현재 세션 내에서 객체를 현제 Session에 저장한다. Session에 저장된 객체에서 DB에 하는 모든 요청은 Session cache를 거친다. Session 객체가 활성중일 때만 Session cache를 사용할 수 있다.
6. **Transaction** 
    - 데이터의 영속성을 달성시키고(commit) 예기치 못한 상황이 발생할 경우 롤백할 수 있게한다.
7. **Persistent objects** 
   - Persistent objects는 관계형 데이터베이스의 한 필드로서 유지되는 평범한 오래된 Java 객체들(POJOs)이다.
   - 구성 파일(hibernate.cfg.xml 또는 hybernate.properties)에서 구성하거나 @Entity 어노테이션으로 선언될 수 있다.
8. **Second-level-cache** : 
   - 여러 세션에 걸쳐 객체를 저장하는 데 사용된다(application 단위의 cache). 
   - Second-level-cache는 명시적으로 사용되며, 해당 캐시에 대한 캐시 공급자를 제공해야 한다. 
   - 일반적인  Second-level-cache 공급자 중 하나로 EhCache가 있다.

___

## 💿 Spring-Data

![image](https://suhwan.dev/images/jpa_hibernate_repository/overall_design.png)

> Spring에서 제공하는 모듈 중 하나로 개발자가 더 쉽게 사용할 수 있도록 JPA를 한단계 더 추상화 시킨 Repository라는 interface를 제공

**즉, Spring-Data-JPA의 Repository의 내부적인 구현에서 JPA를 사용한다는 뜻**

___

## 💿 JDBC

> Java Database Connectivity의 줄임말. java에서 DB에 접속할 수 있도록 하는 API

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile27.uf.tistory.com%2Fimage%2F992473455ACDBAD8118620)	

- JDBC Driver Manager는 DB에 맞는 Driver를 load하여 JDBC를 초기화 합니다
- JDBC Driver들은 Java App의 요청들을 DBMS가 이해 할 수 있는 Protocal로 변환합니다.

![image](https://gmlwjd9405.github.io/images/spring-framework/spring-jdbc-architecture.png)
