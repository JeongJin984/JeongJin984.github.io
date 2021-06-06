---
title: Functional programming1
author: Jiny
date: 2021-03-05 22:42:00 +0800
categories: [programming]
tags: [functional, basic]
toc: false
---

# 함수형 프로그래밍(Functional Programming)
___

## 🔘 함수형 프로그래밍

> 함수를 1급 객체로 취금하여 프로그래밍 하는 형식

### 1급 객체(First Object)

- 변수나 데이터 구조안에 담을 수 있다.
- 파라미터로 전달 할 수 있다.
- 반환값으로 사용할 수 있다.
- 할당에 사용된 이름과 관계없이 고유한 구별이 가능하다.
- 동적으로 프로퍼티 할당이 가능하다.

### 고차함수(High-Order Function)

> 람다 계산법에 의해 아래의 조건을 만족하는 함수

- 함수에 함수를 파라미터로 전달할 수 있다.
- 함수의 반환값으로 함수를 사용할 수 있다.

**Plus**

- 고차함수는 1급 함수의 부분집합이다.

___

## 🔘  불변성

- 함수형 프로그래밍에서는 데이터가 변할 수 없는데, 이를 불변성이라고 한다.
- 데이터 변셩이 필요한 경우 원본 데이터 구조를 변경하지 않고 데이터를 복사본을 만들어 그 일 부를 젼경하고, 변경한 복사본을 사용해 작업을 진행

___

## 🔘 순수 함수

- 동일한 입력에는 항상 같은 값을 반환해야 한다.
- 함수의 실행은 프로그램의 실행에 영향을 미치지 않아야 한다.(Side-Effect가 없어야 한다.)
___

## 🔘 데이터 변환

- 함수형 프로그래밍은 데이터의 변경이 불가하기 때문에 기존 데이터의 복사본을 만들어 주는 도구들이 필요
- ex) js: Array.map, Array.reduce 등 데이터 복사본을 만들기 위한 함수들을 제공

___

## 🔘 합성 함수

- 둘 이상의 함수를 조합하는 과정
- 함수들은 연쇄적으로 또는 병렬로 호출해서 더 큰 함수를 만드는 과정

### 메서드 체이닝

```javaScript
const sum = (a, b) => a + b
const square = x => x * x
const addTen = x => x + 10

const computeNumbers = addTen(square(sum(3, 5))) // 74

// compose는 함수를 연쇄적으로 호출하면서 반환값을 전달한다 
const compose = (...fns) =>
  fns.reduce((prevFn, nextFn) =>
    (...args) => nextFn(prevFn(...args)),
  );

// compose의 사용
const compute = compose(
  sum,
  square,
  addTen
)
compute(3, 5) // 74
```

## 🔘 SO...

> 함수형 프로그래밍은 순수함수(pure function)을 조합하고 공유상태(shared state), 변경 가능한 데이터 및 부작용(side-effects)을 피하여 소프트웨어를 만드는 프로세스다.

> 함수형 프로그래밍은 명령형(imperative)이 아닌 선언형 이며 app의 상태는 순수 함수를 통해 전달된다.
