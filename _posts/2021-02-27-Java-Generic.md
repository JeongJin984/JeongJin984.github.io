---
title: Java Generic
author: Jiny
date: 2021-01-27 22:40:00 +0800
categories: [Java, Jasic]
tags: [java, basic]
toc: false
---

# Generic
___

## 🔘 Generic

> Data Type을 일반화 하는것을 의미

- Classs 나 Method에서 사용할 내부 데이터 타입을 컴파일 시에 미리 지정하는 방법
  - Class나 메서드 내부에서 사용되는 객체의 타입의 안정성을 높일 수 있다.
  - 반환값에 대한 타입 변환 및 타입 검사에 들어가는 노력을 줄일 수 있다.
- Object 타입으로 사용시 다시 타입 변환을 해줘야 하는 번거로움이 있을 수 있다.

### 장점

- 잘못된 타입이 사용될 수 있는 문제를 컴파일 과정에서 제거할 수 있기 때문
- 컴파일러는 제네릭 코드에 대해 강한 타입 체크를 한다.
- 실행 시 타입 에러가 나는 것보다 컴파일 시에 미리 타입을 강하게 체크해서 에러를 사전에 방지
- 타입 변환을 할 필요가 없어 프로그램 성능이 향상

### 사용

```java
class GenericSample<T> {
	T element;
	void setElement(T element){
		this.element = element;
		T getElement() {
			return element;
		}
	}
}
```

### 타입 변수

- 이름에 제한 없음
- 여러개의 타입 변수는 ,로 구분
- Class에서 뿐만 아니라 Method의 매개변수나 반환값으로도 사용 가능
- 임의의 참조형 타입을 의미
- 현존하는 클래스를 사용해도 되고 존재하지 않는 것을 사용해도 됨

### 이름

- E: 요소
- K: 키
- N: 숫자
- T: 타입
- V: 값
- S,U,V: 두번쨰, 세번째, 네번째

___

## 🔘 Bounded type parameter

> 특정 타입의 서브 타입으로 제한

```java
public class BoundTypeSample<T extends Number> {
	public void set(T value) {}
	public static void main(String[] args) {
		BoundTypeSample<Integer> bound = new BoundTypeSample<>();
	}
}
```

- Number의 서브 타입만 허용

## 🔘 WildCard

> 제네렉으로 구현된 Method의 경우 선언된 타입으로만 매개변수를 입력해야 한다. 이를 상속받은 클래스 혹은 부모 Class에서 사용 불가하기 하고 어떤 타입이 와도 상관없는 경우에 대응하기 좋지 않다.

### Unbounded WildCard

- List<?> 와 같은 형태로 물음표만 가지고 정의
  - 내부적으로 Object로 정의되어서 사용되고 모든 타입의 인자를 받을 수 있다.
- Object 클래스에서 제공되는 기능을 사용하여 구현할 수 있는 Method를 작성할 경우
- 타입 파라미터에 의존적이지 않은 Class의 Method를 사용하는 경우

### Upper Bounded WildCard

- List<? extends Foo> 와 같은 형태로 특정클래스의 서브 클래스만 인자로 받음
  - Foo 클래스에 정의된 기능만 사용이 가능하다.

### Lost Bounded WildCard

- List<? super Foo> 와 같은 형태로 특정 클래스의 상위 클래스만 인자로 받음

___

## 🔘 매개변수화 타입(Parameterized Type)

- 하나 이상의 타입 매개변수를 선언하고 있는 Class 나 Interface를 제네릭 클래스 또는 제네릭 인터페이스라고 하고 이를 제네릭 타입이라 한다. 각 제네릭 타입에서는 매개변수화 타입들을 정의한다.
  

```java
List<String>list = new ArrayList<>();
```

- <>안의 String은 실 타입 매개변수라고 하고 List 인터페이스에 선언되어있는 List의 E를 형식 타입 매개변수라고 한다. 
  - 제네릭은 타입소거자에 의해 자신의 타입 요소 정보를 삭제한다.
  - ArrayList를 생성할 때 어떠한 타입 정보도 들고 있지 않게 된다.
  - 즉 new ArrayList() 와 같음
- 컴파일러는 컴파일 단계에서 List 컬렉션에 String 인스턴스만 저장되어야 한다는 것을 알게되었고 또 그것을 보장해주기 때문에 ArrayList list로 변경하여도 런타임에 동일한 동작을 보장
- 타입 소거자에의해 컴파일 타임에 타입 정보가 사라지는 것: 비 구체화
  - E, List 와 같은 런타임에 구체화하지 않는 타입들을 비 구체화(non reifiable type)타입이라고 한다.
  - 반대로 primitive, non-generic types, raw types 또는 List<?> Map 과 같은 Unbounded WildCard Type은 런타임에 구체화 하여 구체화(reifiable type)이라고 한다.

### 제네릭 선엔에 사용하는 타입의 범위도 지정 가능

```java
public class CarWildCardSample {

	public static void main(String[] args) {
		CarWildCardSample sample = new CardWukdcardSample();
		sample.callBoundedWildcardMethod();
	}

	public void callBoundedWildcardMethod() {
		WildcardGeneric<Car> wildcard = new WildcardGeneric<Car>();
		boundedWildCardMethod(wildcard);
	}
	
	public void boundedWildCardMethod(WildCardGeneric<? extends Car> c) {
		Car value = c.getWildCard();
	}
}
```

```java
public class WildcardGeneric<W> {
	W wildcard;
}
```

```java
public class Car {}

public class Bus extends Car {}
```
___

## 🔘 제네릭 메서드

```
- 접근지시자 <타입파라미더, ...> 리던타입 메서드명(매개변수, ...) { ... }의 형식
- public <T> Box<T> boxing(T t) { ... };
```

### 호출

1. Box<Integer> box = <Integer> boxing(100);
	- 명시적으로 구체적 타입 지정
2. Box<Integer> box = boxing(1000);
	- 매개값을 보고 구체적 타입을 추정 

___

## 🔘 Erasure

> 원소 타입을 컴파일 타임에서만 검사하고 런타임에는 해당 타입 정보를 알수가 없다.

- 즉 컴파일 상태에만 제약 조건을 적용하고 런타임에는 타입에 대한 정보를 소거

### 과정

- 제네릭 타입에서는 해당 타입 파라미터(T)나 Object(unbounded)로 변경
- 타입 안정성 보존을 위해 필요시 type casting 삽입
- 확장된 제네릭 타입에서 다형성 보존하기 위해 bridge method 생성

### 유형

### Class Type Erasure

클래스 수준에서 컴파일러는 클래스의 Type Parameter를 버리고 첫번째 바인딩으로 대체하거나 Type Parameter가 바인딩(extends 등으로) 되지 않은 경우 Object로 변환

### Method Type Erasure

method-level type erasure가 바인딩되지 않은 경우 부모 형식 Object로 변환되거나 바인딩 될 때 첫번쨰 바인딩 된 클래스로 변환