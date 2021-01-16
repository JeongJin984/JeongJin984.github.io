---
title: Java Exception
author: Jiny
date: 2021-01-16 11:48:00 +0800
categories: [Java, Jasic]
tags: [java, basic]
toc: false
---

# Java Exception
___

## Interface

>자바의 다형성을 극대화하여 개발코드 수정을 줄이고 프로그램 유지보수성을 높이기 위해 인터페이스를 사용

## Interface 정의

```java
public interface 인터페이스명 {

//상수
타입 상수명 = 값;

//추상 메소드
타입 메소드명(매개변수, ... );
 
//디폴트 메소드
default 타입 메소드명(매개변수, ... ){
  //구현부
}

//정적 메소드
static 타입 메소드명(매개변수) {
  //구현부
}
```

- 상수 : 인터페이스에서 값을 정해줄테니 함부로 바꾸지 말고 제공해주는 값만 참조 (절대적)
- 추상메소드 : 가이드 제공, 추상메소드를 오버라이팅해서 재구현 (강제적)
- 디폴트메소드 : 인터페이스에서 기본적으로 제공, 맘에 안들면 각자 구현 (선택적)
- 정적메소드 : 인터페이스에서 제공해주는 것으로 무조건 사용 (절대적)

## 인터페이스 구현

```java
public interface Bank {
 
    //상수 (최대 고객에게 인출해 줄 수 있는 금액 명시)
    public int MAX_INTEGER = 10000000;
     
    //추상메소드(인출하는 메소드)
    void withDraw(int price);
     
    //추상메소드(입금하는 메소드)
    void deposit(int price);
     
    //JAVA8에서 가능한 defualt 메소드(고객의 휴면계좌 찾아주는 메소드 : 필수구현은 선택사항)
    default String findDormancyAccount(String custId){
        System.out.println("**금융개정법안 00이후 고객의 휴면계좌 찾아주기 운동**");
        System.out.println("**금융결제원에서 제공하는 로직**");
        return "00은행 000-000-0000-00";
    }
     
    //JAVA8에서 가능한 정적 메소드(블록체인 인증을 요청하는 메소드)
    static void BCAuth(String bankName){
        System.out.println(bankName+" 에서 블록체인 인증을 요청합니다.");
        System.out.println("전 금융사 공통 블록체인 로직 수행");
    }    
}
```

```java
public class KBBank implements Bank{
 
    @Override
    public void withDraw(int price) {
        System.out.print("KB은행만의 인출 로직...");
        if(price < Bank.MAX_INTEGER){
            System.out.println(price+" 원을 인출한다.");  
        }else{
            System.out.println(price+" 원을 인출실패.");  
        }
    }
 
    @Override
    public void deposit(int price) {
        System.out.println("KB은행만의 입금 로직..."+price+" 원을 입금한다.");
     
    }
 
}
```

## Interface 와 abstract Class

### 차이점

1. 추상 클래스는 일반 메서드와 추상 메서드 둘다 가질 수 있다.
    - 인터페이스는 오로지 추상 메서드와 상수만을 가진다. (구현 로직을 작성할 수 없다.)
    - **but**. java8 부터는 **default method**와 **static method**를 작성할 수 있다.
2. 인터페이스 내에 존재하는 **메서드**는 무조건 **"public abstract"** 로 선언되며, 이를 생략할 수 있다.
3. 인터페이스 내에 존재하는 **변수**는 무조건 **"public static final"** 로 선언되며, 이를 생략할 수 있다.
4. Java는 상속은 하나만 implement는 여러개 할 수 있다.

```java
// interface 의 제약 조건을 따르지 않았기 때문에 오류가 발생합니다.
private int a = 1;       

// 컴파일러가 자동적으로 public static final b = 2로 추가해줍니다.
public int b = 2;        

// 컴파일러가 자동적으로 public static final c = 3으로 추가해줍니다.
static int c = 3;         

// 컴파일러가 자동적으로 public static final d = 4로 추가해줍니다.
int d = 4;         
```

## 익명 구현 객체

> 구현 객체를 만드는 것이 일반적이나 **일회성 객체**를 위해 소스코드를 생성하고 인터페이스를 정의 하는 것은 비효율적 -> 익명 구현 객체

```java
인터페이스 변수 = new 인터페이스(){
	// 인터페이스에 선언된 추상 메소드의 실체 메소드 정의
}
```

## Interface 상속, 구현체 사용

- 서브 인터페이스는 수퍼 인터페이스의 메서드까지 모두 구현해야 함
- 인터페이스 레퍼런스는 인터페이스를 구현한 클래스의 인스턴스를 가리킬 수 있고,
해당 인터페이스에 선언된 메서드(수퍼 인스턴스 메소드 포함)만 호출 할 수 있다.


## Interface 다중 상속

> Interface는 Class와 달리 다중 상속 가능

 - 추상 메서드로 구현하기 전의 메서드이기 떄문에 어떤 인터페이스의 메서드를 상속받아도 같기 때문
- 메서드 명과 파라미터 형식은 같지만 리턴 타입이 다른 메서드가 있다면, 둘중 어떤 것을 상속받느냐에 따라 규칙이 달라지기 때문에 다중 상속이 불가능   

```java
public interface Bar extends Foo {
    void barMethod();
}
```

```java
public interface Foo {

    void fooMethod();
}
```

```java
public class DefaultFoo implements Bar {
    @Override
    public void barMethod() {
        System.out.println("bar method - by DefaultFoo");
    }

    @Override
    public void fooMethod() {
        System.out.println("foo method - by DefaultFoo");
    }

    public void testMethod_1(){
        System.out.println("test method _ 1");
    }
    public void testMethod_2(){
        System.out.println("test method _ 2");
    }
}
```

```java
public class App {
    public static void main(String[] args){
        System.out.println("hi");

        DefaultFoo defaultFoo = new DefaultFoo();
        defaultFoo.fooMethod();
        defaultFoo.barMethod();
        defaultFoo.testMethod_1();
        defaultFoo.testMethod_2();

        Bar bar = defaultFoo;
        bar.barMethod();
        bar.fooMethod();
    }
}
```

- Class Type 레퍼런스:  해당 클래스에 정의된 메서드 호출 가능
- Interface Type 레퍼런스: 해당 Interface를 구현한 클래스 인스턴스를 가르킬 수 있고 해당 Interface에 선언된 메서드만 호출할 수 있음 

## 다중 Interface에서의 Interface Reference

> 한 클래스가 여러 개의 인터페이스를 구현했다면 각 인터페이스로 구분해서 그 객체를 사용 가능

- 구현체를 어떤 Interface Reference에 담느냐에 따라 사용할 때 따르는 규칙이 달라짐

```java
public class App {
    public static void main(String[] args){
        System.out.println("hi");

        DefaultFoo defaultFoo = new DefaultFoo();
        defaultFoo.fooMethod();
        defaultFoo.barMethod();

        Bar bar = defaultFoo;
        bar.barMethod();
        bar.fooMethod();

        Baz baz = defaultFoo;
        baz.bazMethod();
    }
}
```

## Interface의 Default Method

- 인터페이스에 메소드 선언이 아니라 **구현체**를 제공하는 방법
- 해당 인터페이스를 구현한 클래스의 어떠한 영향없이 새로운 기능을 추가하는 방법
- default method 는 해당 인터페이스를 구현한 **구현체가 모르게 추가된 기능임으로 그만큼 리스크가 따른다.**
    - 컴파일 에러는 발생하지 않지만, 특정한 구현체의 로직에 따라 런타임 에러가 발생할 수 있다.
    - 사용하게 된다면, 구현체가 잘못사용하지 않도록 반드시 문서화 하자!
- Object가 제공하는 기능(equals, hashCode)와 같은 기본 메소드는 제공할 수 없다.
    - 구현체가 재정의 하여 사용하는 것은 상관없다.
- 본인이 수정할 수 있는 인터페이스에만 기본 메소드를 제공할 수 있다.
- 인터페이스를 상속받은 인터페이스에서 다시 추상 메소드로 변경할 수 있다.
- 인터페이스 구현체가 default method를 재정의 할 수 있다.

## static method

> 해당 인터페이스를 구현한 모든 인스턴스, 해당 타입에 관련되어 있는 유틸리티, 헬퍼 메소드를 제공하고 싶다면? → static method로 제공

- 인스턴스 없이 수행할 수 있는 작업을 정의

## Java 9

> java9 에서는 추가적으로 private method 와 private static method 가 추가

- 단지 특정 기능을 처리하는 내부 method일 뿐인데도, 외부에 공개되는 public method로 만들어야 하기 때문
- 코드의 중복을 피하고 interface에 대한 캡슐화를 유지