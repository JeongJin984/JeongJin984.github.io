---
title: Java Lambda
author: Jiny
date: 2021-03-05 22:40:00 +0800
categories: [Java, Jasic]
tags: [java, basic]
toc: false
---

# Lambda
___

## 함수형 프로그래밍

> 함수를 1급 객체로 취금하여 프로그래밍 하는 형식

- 자세한 것은 Funtional Programming 참고

___

## 🔘 Lambda Expressions

> 식별자 없이 실행 가능한 함수(()->{return foo;} or ()-> ())

- 메서드를 하나의 식으로 표현하는 것
- 익명함수(anonymous function)이라고도 표현

### 장점

- 가독성
- 간결성
- 멀티쓰레드 환경에 용이
- 함수를 만드는 과정없이 한번에 처리 -> 생산성

### 단점

- 재사용 불가
- 디버깅이 어렵다
- 무분별한 사용은 가독성 저하
- 재귀 사용 불가

___

## 🔘 @FunctionalInterface

- Lambda 재사용을 위해 변수에 저장이 가능
- 매개변수 혹은 반환값에 따라 함수의 자료형이 달라진다.

**Interfaces**

![image](https://i0.wp.com/javatechonline.com/wp-content/uploads/2020/05/image6.jpg?resize=640%2C478&ssl=1)

___

## 🔘 Variable Capture

- Lambda는 특정 상황에서 람다 함수 본문 외부에 선언된 변수에 접근이 가능하다.
- java8 버전 이전에는 익명의 내부 클래스가 이를 둘러싼 메서드에 대한 로컬 변수를 캡처할 때 문제가 발생했다.
  - 컴파일러가 만족할 수 있도록 로컬 변수 앞에 final 키워드를 추가해야 했다.
- 우리가 변수를 final로 선언하면 컴파일러가 변수를 사실상 final로 인식 가능

### Local variable Capture

- Local variable은 무조건 final 혹은 effective final 이어야 한다.
  - effective Final 이란 syntactic sugar의 일종으로 초기화 이후 값이 한번도 변경되지 않는다는 것
  - 또한 final 키워드가 붙어있지 않지만 final 키워드를 붙인 것과 동일하게 컴파일에서 처리한다.
  - 결론적으로 초기화하고 값이 변경되지 않는 것을 의미한다.

### WHY?

> JVM의 메모리 구조를 살펴보자

- 지역변수를 쓰레드끼리 공유가 안된다.
- JVM에서 인스턴스 변수는 힙영역에 생성된다.
- 인스턴스 변수는 쓰레드 끼리 공유가 가능하다.

**즉** 지역변수가 스택에 저장되기 떄문에 람다식에서 값을 바로 참조하는것에 제약이 있어 복사된 값을 사용하는데 

이때 멀티 쓰레드 환경에서 변경이 되면 동시성에 대한 이슈에 대응하기 힘들기 때문

함수형 프로그래밍에서 함수는 순수함수로만 구성되어야 하며 side-Effect에 대응해야 한다.

```java
Supplier<Integer> incrementer(int start) {
    return () -> start++;
}
```

위의 코드는 컴파일 되지 않는다.
start는 지역 변수이고 final이 아니다.

- 자바에서 람다는 매개변수가 가비지 컬렉터에 수집될 때 까지 실행되지 않는다. 
- 즉 자바는 복사본을 만들어야 한다.

___

## 🔘 Method Reference

- 메서드를 간결하게 지칭할 수 있는 방법으로 람다가 쓰이는 곳에서 사용이 가능
- 일반 함수를 람다 형태로 사용할 수 있도록 함

### 생성자 참조
```java
String::new // 매개변수가 있어도 적합한 생성자를 유추한다.
() -> new String() // 위 코드와 같은 의미
```

### 스태틱 메서드 참조

```java
String::valueOf
x -> String.valueOf(x) // 위와 동일
```

### 인스턴스 참조

```java
x::toString // x 는 접근하고자 하는 인스턴스
() -> x.toString() // 위와 동일
```