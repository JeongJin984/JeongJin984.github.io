---
title: Java Class
author: Jiny
date: 2020-11-28 14:30:00 +0800
categories: [Java, Jasic]
tags: [java, basics]
toc: false
---

# Java Class
___

## 클래스를 정의하는 방법

1. 클래스의 구조
  - 시그니처
    - 다른 클래스로 확장(Extends) 선언 가능 (슈퍼 클래스, 서브 클래스 관계 설정)
    - 확장(Extends) 시, 클래스의 멤버들 접근을 제어할 수 있음 (public, protected, private 등)
    - 다른 클래스를 인터페이스(Interface)로 선언 가능
  - 멤버 변수: Class 안의 기능을 끄집어 낼 때 사용
  - 메서드: 기능을 나타냄
  - 생성자: 처음에 초기화 해줄 값

2. 클래스의 선언

```java
public class Practice{

	public static void main(String[] args){
    	Person junseo = new Person(); // Person이라는 타입으로 메모리 요청. 이 구문이 실행시 Person 클래스의 생성자가 실행됨
        
        Person iu = new Person("iu", 27); // 매개변수가 있는 생성자가 호출됨.
        
        iu.hello(); // Person 클래스의 hello() 메소드가 실행됨(1번 메소드)
        iu.hello("junseo"); // 2번 메소드
        System.out.println(iu.hello1()); // 3번 메소드
        System.out.println(iu.hello1("junseo")); // 4번 메소드
    }
}
```

3. 메서드의 4가지 형태
  - 화면에 출력만 하고 끝나는 기능(ex. hello())
  - 매개변수를 받는 경우(ex. hello(String to))
  - return 값을 돌려주는 경우(ex. hello1())
    - 메소드 타입과 리턴 값의 타입을 맞춰줘야함
  - 매개변수를 받아서 return 하는 경우(ex. hello1(String to))

## 클래스의 생성

> new 란 클래스로 부터 객체를 생성시키는 연산, 힙(Heap) 영역에 저장할 공간을 할당해서 참조 값을 객체에게 반환

- 클래스: 객체의 BluePrint
- 객체: 실제로 생성된 클래스(생성 = Instance화)

## 메서드 정의

> 메서드란 어떠한 문제를 처리하기 위한 방법을 소스 코드로 묶어놓고 필요(호출)에 따라 동작하는 기능

```java
public class MyClass{
    public void method1(){
        System.out.println("method1이 실행됩니다.");
    }
}
```

## 생성자의 정의

```java
public class MyClass{
    public MyClass(){
        System.out.println("method1이 실행됩니다.");
    }
}
```

- 생성자는 접근 제한 수식어인 public, protected, private 만 가능
- 클래스에 생성자가 없을 경우, 기본 생성자가 자동으로 추가
- 클래스의 이름과 함수명이 같아야 함
+ (private 일 경우)외부에서 객체(instance)를 생성하는 과정을 통제하는 수단.

## 기본 생성자

> 자바는 클래스의 인스턴스가 생성될 때 마다 클래스의 생성자 메소드가 호출될 것을 보장한다. 또한 모든 서브 클래스의 인스턴스가 생성될 때마다 수퍼클래스의 생성자가 호출될 것을 보장한다.

- 이를 위해서 생성자의 첫번째 명령문이 this()나 super를 사용하여 다른 생성자를 명시적으로 호출하지 않는다면 super() 호출을 암시적으로 삽입한다.
  - 즉 인자없는 생성자를 가지고 있지 않다면 이러한 암시적 호출을 컴파일 에러를 유발한다.
- 상속 관계에서 모든 클래스는 Object 클래스를 상속하므로 기본적으로 모든 최상위 부모 클래스의 생성자는 super()를 암묵적으로 가진다.

## this 키워드

- 현재 클래스의 인스턴스를 의미
- 즉, 현재 클래스의 멤버변수를 지정할때 사용
- 동일한 이름의 파라미터가 들어올 경우, 명확히 구분

## 생성자 체이닝

![이미지](https://beginnersbook.com/wp-content/uploads/2013/12/constructor_chaining.jpg)*생성자 체이닝*

## 초기화 블럭

**자바에서 초기화 순서**

1. 클래스 변수 : 기본값 → 명시적 초기화 → 클래스 초기화 블록
2. 인스턴스 변수 : 기본값 → 명시적 초기화 → 인스턴스 초기화 블록 → 생성자

```java
public class Square {
    int x;
    int y;
    boolean b;
}
```
- 이 경우 모든 필드 들은 primitive의 경우 0으로 reference의 경우 null로 초기화
- 초기화 블럭은 초기화 과정에서 로직을 추가하고 싶을 경우 사용

```java
class InitBlock {

    static int classVar = 10;         // 클래스 변수의 명시적 초기화

    int instanceVar = 10;             // 인스턴스 변수의 명시적 초기화

    static { classVar = 20; }         // 클래스 초기화 블록을 이용한 초기화

    { instanceVar = 20; }             // 인스턴스 초기화 블록을 이용한 초기화

    InitBlock() { instanceVar = 30; } // 생성자를 이용한 초기화

}

 

public class Member05 {

    public static void main(String[] args) {

        System.out.println(InitBlock.classVar);

        InitBlock myInit = new InitBlock();

        System.out.println(myInit.instanceVar);

    }

}
```

> 실행결과

```
20
30
```

(static 블록 = 클래스 초기화 블록)