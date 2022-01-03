---
title: Spring AOP
author: Jiny
date: 2021-11-25 13:09:00 +0800
categories: [Java, Spring]
tags: [java, spring, aop]
toc: false
---

# Spring AOP
___

## 💿 프록시, 프록시 패턴

![image](https://t1.daumcdn.net/cfile/tistory/99A46433599FE0A41E)

### 주요 기능
- 접근 제어
  - 권한에 따른 접근 차단
  - 캐싱
  - 지연 로딩
- 부가 기능 추가
  - 원래 서버가 제공하는 기능에 더해서 부가 기능을 수행한다.
  - 예) 요청 값이나, 응답 값을 중간에 반영한다.
  - 예) 실행 시간을 측정해서 추가 로그를 남긴다.

### 데코레이터 패턴과 차이
- 둘다 프록시를 사용하는 방법이지만 GOF 디자인 패턴에서는 이 둘을 의도(intent)에 따라서 나눈다.
  - 프록시: 접근 제어가 목적
  - 데코레이터: 새로운 기능 추가


```java
    @Slf4j
    public class CacheProxy implements Subject {
        private Subject target;
        private String cacheValue;

        public CacheProxy(Subject target) {
            this.target = target;
        }

    @Override
    public String operation() {
        log.info("프록시 호출");
        if (cacheValue == null) {
            cacheValue = target.operation();
        }
        return cacheValue;
        }
    }
```

### 프록시 종류

- 인터페이스 기반 프록시
  - 인터페이스를 상속한 RealSubject가 있고
  - 인터페시를 상속하고 프록시 대상 객체를 인터페이스로서 가진 Proxy 객체
- 구체 클래스 기반 프록시
  - 클래스를 상속하고 프록시 대상 객체를 가진 Proxy 객체로 구현
- 따라서...
  - 인터페이스가 없어도 클래스 기반으로 프록시를 생성할 수 있다.
  - 클래스 기반 프록시는 해당 클래스에만 적용할 수 있다.
  - 인터페이스 기반 프록시는 인터페이스만 같으면 모든 곳에 적용할 수 있다.
  - 클래스 기반 프록시는 상속을 사용하기 때문에 제약이 몇몇 존재
    - 부모 클래스의 생성자를 호출해야 한다.
    - 클래스에 final 키워드가 붙으면 상속이 불가능하다.
    - 메서드에 final 키워드가 붙으면 해당 메서드를 오버라이딩 할 수 없다.

___

## 동적 프록시

### 리플렉션

> 리플렉션은 구체적인 클래스 타입을 알지 못해도, 그 클래스의 메소드, 타입, 변수들에 접근할 수 있도록 해주는 자바 API

- 클래스들은 런타임 시점에 클래스 로더에 의해 Heap 영역에 저장된다.
- 저장된 객체들은 실제로 new가 되어 저장되어 있지 않기 때문에 구체적인 클래스를 알 수 없다.