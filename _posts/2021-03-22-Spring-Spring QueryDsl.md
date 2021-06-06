---
title: Spring QueryDsl
author: Jiny
date: 2021-03-22 20:30:00 +0800
categories: [Java, Spring]
tags: [java, spring, jpa]
toc: false
---

# Spring QueryDsl
___

## 💿 기본 문법

### 기본 검색 Query

```java
Memer findMember = queryFactory
      .selectFrom(member)
      .where(member.username.eq("member1")
               .and(member.age.eq(10)))
      .findOne();
```

- 검색 조건은 .and() 혹은 .or()를 메서드 체인으로 연결할 수 있다.
- selectFrom은 select() 와 from()을 합친 것 


### 결과 조회

- fetch(): 리스트 조회, 데이터 없으면 빈 리스트 반환
- fetchOne(): 단 건 조회
  - 결과가 없으면 null
  - 결과가 둘 이상이면: NonUniqueResultException
- fetchFirst(): limit(1).fetchOne()
- fetchResults(): 페이징 정보 포함, total count 쿼리 추가 실행 
- orderby()로 sortinf 가능