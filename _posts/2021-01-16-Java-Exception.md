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

## 💿 **목차**
- 자바에서 예외 처리 방법 (try, catch, throw, throws, finally)
- 자바가 제공하는 예외 계층 구조
- Exception과 Error의 차이는?
- RuntimeException과 RE가 아닌 것의 차이는?
- 커스텀한 예외 만드는 방법

### **예외(Exception)와 에러(Error)**

- 발생시점
    - 컴파일 에러
    - 런타임 에러
- 발생 원인
    - 논리적 에러
    - 시스템 에러

> 에러: 하드웨어의 오작동으로 실행 오류가 발생하는 것

- 메모리 부족, 스택 오버플로우 등과 같이 발생하면 복구할 수 없는 오류
- 발생하더라도 수습될 수 있는 비교적 덜 심각
- 즉 에러는 JVM 실행에 문제가 생긴 것이므로 개발자가 대처하기 어려움

> 예외: 사용자의 잘못된 조작 혹은 개발자의 잘못된 코딩으로 발생하는 프로그램 오류

- 예외는 발생되면 프로그램은 바로 종료된다는 점에서는 에러와 동일
    - 예외는 예외 처리를 통해 프로그램을 종료하지 않고 정상 실행 상태를 유지할 수 있음

___

## 💿 **에러의 종류**

> 모든 예외 클래스는 java.lang.Exception을 상속 받는다.

![image](https://t1.daumcdn.net/cfile/tistory/2626EC4956BBDA832A)

- 일반 예외(컴파일러 에러): 자바 소스를 컴파일 하는 과정에서 생긴 에러
- 실행 예외(런타임 에러): 클래스 파일을 실행시키는 과정에서 생긴 에러

___

## 💿 **예외의 종류**

- 체크 예외: RuntimeException을 상속하지 않은 예외
  - 반드시 예외 처리를 하는 코드를 함께 작성해야 함
  - IOException, SQLException 등등
- 언체크 예외: RuntimeException을 상속한 예외
  - 예외 처리 코드를 강제하지 않음
  - 주로 프로그램의 논리 오류가 있을 떄 발생하도록 의도된 것

### **예외 처리**

**try-catch**
```java
try {
    // 예외가 발생할 가능성이 있는 코드
} catch (Exception1 e1) { // Exception을 상속받는 참조 변수
    // Exception1이 발생했을 때, 이를 처리하기 위한 코드
} catch (Exception2 e2) {
    // Exception2가 발생했을 때, 이를 처리하기 위한 코드
} catch (ExceptionN eN) {
    // ExceptionN이 발생했을 때, 이를 처리하기 위한 코드
}
```

- 발생한 예외와 일치하는 단 한개의 catch 블럭만 실행
- catch 블럭 안에 참조 변수의 이름이 중복되는 안된다.

### **throw**

> 고의로 예외를 발생시키는 키워드

```java
public class ExceptionDemo {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.println("아이디를 입력하세요");
        String userName = scanner.nextLine();

        try {
            if (userName.equals("바보")) {
                throw new IllegalArgumentException("부적절한 이름입니다.");
            }
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

```java
void method() throws Exception1, Exception2, ... ExceptionN {
	// 메서드 내용
}
```

### **finally**

```java
try {
	// 예외가 발생할 가능성이 있는 문장을 넣는다.
} catch {
    // 예외 처리를 위한 문장을 넣는다.
} finally {
    // 예외 발생 여부와 상관없이 항상 실행되어야 할 문장을 넣는다.
}
```

- finally 블록 내의 문장은 try, catch 블록에 return문이 있더라도 실행됨

___

## 💿 **예외 계층 구조**

![image](https://miro.medium.com/max/799/1*3CUfvqPJ--M8uGOjq13uMQ.png)

- Checked Exception: 컴파일 시점에서 확인 할 수 있는 예외
    - 반드시 처리 혹은 메서드 선언부에 예외 선언을 해주어야 한다.
- UnChkeced Exception: 안그런 예외
    - RuntimeException과 그 하위 클래스, Error

### **사용자 정의 Exception**

> Exception 클래스를 상속

```java
class SpaceException extends Exception {
    public SpaceException(String message) {
        super(message);    // 조상 클래스인 Exception의 생성자 호출
    }
}
```
___

## 💿 **예외처리**

### **예외 복구**

> 예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것

- IOException이 발생했다면, 사용자에게 상황을 알려주고 다른 파일을 이용하도록 안내해서 예외상황 해결 가능(첫번째 파일은 원래대로 해놓고)
- 원격 DB 서버에 접속하다 실패했을 경우 MAX_RETRY 만큼 재시도 하는 것(실패 시 리소스 반납, 정리 작업을 해야 한다.)

### **예외처리 회피**

> 예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것

- throw 문으로 선언해서 예외가 발생하면 알아서 던져지게 하거나
- catch 문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는 것
- 예외 처리를 회피하려면 반드시 다른 오브젝트나 메서드가 예외를 대신 처리할 수 있도록 해야 함

### **예외 전환**

> 예외 회피와 비슷하게 예외를 복구해서 정상적인 상태로는 만들 수 없기 때문에 예외를 메서드 밖으로 단지는 것

- 발생한 예외를 그대로 넘기는 것이 아닌 적절한 예외로 전환해서 던진다는 특징이 있음
- 보통 전환하는 예외에 발생한 예외를 담아서 중첩 예외를 만드는 것이 좋음
- 목적
  - 내부에서 발생한 예외를 그대로 던지는 것이 그 예외상황에 대한 적절한 의미를 부여해주지 못하는 경우
  - 예외를 처리하기 쉽고 단순하게 만들기 위해 포장하기 위해서
    - 주로 예외처리를 강제하는 체크 예외를 처리할 수 없을 때 언체크 예외인 런타임 예외로 바꾸는 경우에 사용




