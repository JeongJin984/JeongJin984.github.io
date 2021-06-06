---
title: Java Reflection
author: Jiny
date: 2020-11-28 14:30:00 +0800
categories: [Java, Spring]
tags: [java, reflection]
toc: false
---

# Java Reflection
___

## 리플렉션의 시작은 Class<T>

## Class<T>에 접근하는 방법

- 모든 클래스를 로딩한 다음 Class<T>의 인스턴스가 생긴다.(static 이므로) "Type.class"로 접근할 수 있다.

```java
public class DemoSpringDiApplication {

    public static void main(String[] args) throws ClassNotFoundException {
        Class<Book> bookClass = Book.class;
        System.out.println(bookClass.getName());

        Book book = new Book();
        Class<? extends Book> aClass = book.getClass();
        System.out.println(aClass.getName());

        Class<?> bClass = Class.forName("com.example.demospringdi.Book");
        System.out.println(bClass.getName());
    }
}
```
- 모든 인스턴스는 getClass() 메서드를 가지고 있다. "인스턴스.getClass()"로 접근할 수 있다.
- 클래스를 문자열로 읽어오는 방법
  - Class.forName("FQCN")
  - ClassPath에 해당 클래스가 없다면 ClassNotFoundException이 발생한다.

## Class<T>를 통해 할 수 있는 것

- 필드(목록) 가져오기
- 메서드(목록) 가져오기
- 상위 클래스 가져오기
- 인터페이스(목록) 가져오기
- 애노테이션 가져오기
- 생성자 가져오기
- ...

## Annotation 과 Reflection

**Annotation**
  - 기본적으로 주석과 같은 취급: 소스와 클래스 까지는 남음
    - 로딩 했을 때 메모리 상에 남지 않음(바이트 코드에 남지 않는다.)
    - 따라서 Retention을 사용
  - @Target으로 Annotation 사용 범위를 지정 가능
  - 

**중요 애노테이션**
  - @Retention: 해당 애노테이션을 언제까지 유지할 것인가? 소스 클래스 런타임
  - @Inherit: 해당 애노테이션을 하위 클래스까지 전달 할 것인가
  - @Target: 어디에 사용할 수 있는가?

**Reflection**
  - getAnnotations(): 상속받은 (@Inherit) 애노테이션까지 조회
  - getDeclaredAnnotations(): 자기 자신에만 붙어있는 애노테이션 조회