---
title: Java-Basics
author: Jiny
date: 2020-11-20 10:14:00 +0800
categories: [Lang, Java]
tags: [java, datatype]
toc: false
---

# Java Basics
---

## Data Type

![image](https://lh3.googleusercontent.com/proxy/mBv-_PCXtzdH04OSePTct032YU4ZT3H_ENcQxlNBvzIZ5HHBVm5mMnqJ-6q9q7LSa9nCbygSTGo6M0IvowB-IWcP8Fdg0a7oI53GdB_rlY3nED208N7Sx-SEHlLRo8eeCCA93lN8AXIB6SE8foucHOfk53mYKMbuNQ)

**메모리에 집접 담는가?**
- Primitive Type(기본형): 직접 데이터를 담음
- Reference Type(참조형): 메모리를 참조


## Primitive Type(기본형)

- 총 8가지의 기본형 타입(Primitive type)을 미리 정의하여 제공
- 기본값이 있기 때문에 Null이 존재하지 않는다. 만약 기본형 타입에 Null을 넣고 싶다면 래퍼 클래스를 활용
- 실제 값을 저장하는 공간으로 스택(Stack) 메모리에 저장된다.
- Java에는 C 처럼 Sizeof가 없으나 Primitive Type 클래스가 존재하고 그 안에 각 타입의 크기를 명시

![image](https://loustler.io/files/java_primitive_type.png)
<div style="text-align: center">출처: https://loustler.io/languages/Java-primitive-type/</div>


## Reference Type(참조형)

> 기본적으로 java.lang.Object를 상속

- 클래스형: 객체를 참조 하는 타입
- 인터페이스형: 상수(static final)와 추상 메서드(abstract method)의 집합
  - 생성자를 가질 수 없음
  - 객체화 불가
  - 다중 상속 가능
  - 자바8 부터 인터페이스에 디폴트 메서드(default method)를 지원

## Literal

> 데이터 그 자체


```java
int a = 1;
```

여기서 1이 리터널

## 변수 선언 및 초기화

- 명시적 초기화: 변수 선언과 동시에 초기화
  - int a = 3;
  - ClassA a = new ClassA();
- 초기화 블럭
  - 클래스 초기화 블럭
    - 클래스를 초기화 하는데 사용
    - 클래스가 처음 메모리에 로딩 될 때 한번만 수행
  - 인스턴스 초기화 블럭
    - 인스턴스 변수를 초기화 하는데 사용
    - 생성자와 같이 인스턴스가 생성될 때 수행
    - 인스턴스 초기화 블럭이 생성자보다 먼저 수행

```java
  class A {
    // 클래스 초기화 블럭
    static {
      ...
    }
    // 인스턴스 초기화 블럭
    {
      ...
    }
  }
```

인스턴스 초기화 블럭은 모든 생성자에서 공통으로 수행해야 되는 코드를 삽입

**순서**
- 클래스 변수(static)
  - 기본값 -> 명시적 초기화 -> 클래스 초기화 블럭
- 인스턴스 변수
  - 기본값 -> 명시적 초기화 -> 인스턴스 초기화 블럭 -> 생성자


## 변수의 스코프와 라이프타임

***스코프***

```java
  public class ValableScopeExam{

      int globalScope = 10;   // 인스턴스 변수 

      public void scopeTest(int value){   
          int localScope = 10;
          System.out.println(globalScope);
          System.out.println(localScpe);
          System.out.println(value);
      }
  }
```

- 클래스의 속성으로 선언된 변수 globalScope 의 사용 범위는 클래스 전체
- 매개변수로 선언된 int value 는 블럭 바깥에 존재하기는 하지만, 메서드 선언부에 존재하므로 사용범위는 해당 메소드 블럭 내 
- 메소드 블럭내에서 선언된 localScope 변수의 사용범위는 메소드 블럭 내

***lifetime***

- 전역: app 실행 동안 유지
- 정적(static): app 실행 동안 유지
- 자동/지역: 함수 실행 동안 유지
- 동적: 메모리 해제 전까지 유지


## 타입 변환, 캐스팅 그리고 타입 프로모션

**타입 변환, 캐스팅**

- 묵시적 변환(Promotion)
  - 큰 타입에 저장할 때 큰 타입으로 변환
  - float a = 10;
- 강제 변환(casting)
  - 값 손실 발생
  - char a = (char)65 


## 1차 및 2차 배열 선언하기

- 1차원 배열
  - int []a = new int[10];
  - int a[] = new int[10];
  - int a[] = {1,2,3,4,5}
  - new int[] {1,2,3,4,5} // 매개변수
- 2차원 배열
  - int [][]a = new int[10][10];
  - for() 으로 초기화


## 타입 추론

- ex) var list = new ArrayList<String>();
- initializer가 필수
- 다이아몬드 연산자 사용 불가


## 접근 제어자

> 접근 제어자 - public, protected, default, private <br/>
그 외 - static, final, abstract, transient, synchronized, volatile etc.

- 접근 제어자 : 접근 제어자는 해당 클래스 또는 멤버를 정해진 범위에서만 접근할 수 있도록 통제하는 역할을 한다. 클래스는 public과 default밖에 쓸 수 없다. 범위는 다음과 같다. 참고로 default는 아무것도 덧붙이지 않았을 때를 의미한다.
- static : 변수, 메서드는 객체가 아닌 클래스에 속한다.
- final :
  - 클래스 앞에 붙으면 해당 클래스는 상속될 수 없다.
  - 변수 또는 메서드 앞에 붙으면 수정되거나 오버라이딩 될 수 없다.
- abstract :
  - 클래스 앞에 붙으면 추상 클래스가 되어 객체 생성이 불가하고, 접근을 위해선 상속받아야 한다.
  - 변수 앞에 지정할 수 없다. 메서드 앞에 붙는 경우는 오직 추상 클래스 내에서의 메서드밖에 없으며 해당 메서드는 선언부만 존재하고 구현부는 상속한 클래스 내 메서드에 의해 구현되어야 한다. 상속과 관련된 내용은 6주차에 다룰 예정이다.
- transient : 변수 또는 메서드가 포함된 객체를 직렬화할 때 해당 내용은 무시된다.
- synchronized : 메서드는 한 번에 하나의 쓰레드에 의해서만 접근 가능하다.
- volatile : 해당 변수의 조작에 CPU 캐시가 쓰이지 않고 항상 메인 메모리로부터 읽힌다.
- strictfp: 
  - 플랫폼에 상관없이 부동 소수점 연산에 일관성을 부여한다.
  - 클래스, 메소드, 인터페이스에 사용이 가능하다.