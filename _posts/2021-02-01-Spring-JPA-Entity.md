---
title: Spring-JPA-Entity
author: Jiny
date: 2021-02-01 13:09:00 +0800
categories: [Java, Spring]
tags: [java, spring, jpa]
toc: false
---

# Spring JPA
___

## Entity 설계시 주의사항

**🔘 Entity에는 가급적 Setter를 사용하지 말자**
- 변경 포인트가 많아서 유지보수가 어렵다.


**🔘 모든 연관관계는 LAZY(지연) LOADING으로 설정**
- Eager(즉시) Loading은 예측이 어렵고 어떤 SQL이 실행 될지 추적하기가 어렵다.
  - 특히 JPQL을 실행하면 N+1 문제가 자주 발생한다.(select o from object)(무조건 같이 로딩되기 때문에)
- 연관된 Entity를 함께 DB에 조회해야 한다면, fetch join 또는 Entity Graph를 사용한다.
- OnetoOne, ManyToOne 관계는 기본이 즉시로딩이므로 직접 지연로딩으로 설정


**🔘 컬렉션은 필드에서 초기화 하자**
- null 문제에서 안전하다
- Hibernate는 Entity를 영속화 할 떄 컬렉션을 감싸서 Hibernate가 제공하는 내장 컬렉션으로 변경
  - 만약 getOrders()처럼 임의의 메서드에서 컬렉션을 잘못 생성하면 내부 메커니즘에 문제가 발생할 수 있다.


**🔘 테이블 컬럼명 생성 전략**
- Hibernate 기본전략: Entity의 필드명을 그대로 테이블 명으로 사용(SpringPhysicalNamingStrategy)
- Spring Boot 신규 설정
  1. 카멜 케이스 → 언더스코어(memberPoint → member_point)
  2. .(점) → _(언더스코어)
  3. 대문자 → 소문자

**🔘 Cascade 옵션**
- 영속성 상태를 전파 시키는 옵션
- List에 들어가는 모든 객체를 따로 영속화 할 필요없이 자동으로 영속화


