---
title: Java Inheritance
author: Jiny
date: 2020-12-26 11:19:00 +0800
categories: [Java, Jasic]
tags: [java, basics]
toc: false
---

# Inheritance
___

## 상속의 특징

- 다중 상속 불가
- 자식 클래스는 부모 클래스로부터 메소드와 필드를 물려받아 사용 
- 부모 클래스는 자식 클래스에서 정의한 메소드나 필드를 사용불가
- 부모 클래스는 여러 자식 클래스에게 상속이 가능
- 자바의 모든 클래스는 최상위 클래스 Object의 서브클래스이다.
- 
## super

> 자식 클래스의 객체가 생성될 때, 자동으로 부모 클래스의 생성자(super())가 실행 

- 자식 클래스의 생성자에 super()를 안써주면,자바는 부모 클래스의 기본 생성자를 탐색

## 오버라이딩

> 상속관계에 있는 클래스간에 같은 이름의 메서드를 재정의하는 문법

## abstract class

> 추상 메서드를 포함하고 있다는 것을 제외하고는 일반 클래스와 전혀 다르지 않다.

## final 키워드

- 로컬 원시 변수: final로 선언하면 한번 초기화된 변수 상수화
- 객체 타입: 변수에 다른 참조 값을 지정 불가
- 메서드 인자: 메서드 안에서 변수값을 변경 불가
- 멤버 변수: 생성자 메서드가 끝나기 전에 초기화 되면 상수 값이 되거나 write-once 필드로 한번만 쓰이게 됨
- 메서드: 메서드를 final로 선언하면 상속받은 클래스에서 오버라이드가 불가능
- 클래스: 클래스에 final을 선언하면 상속 불가

## Object 클래스

> 자바 API의 모든 클래스와 사용자가 정의한 모든 클래스의 최상위 클래스

**주요 메서드**
![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbe197E%2FbtqQPz6HMyE%2F7mhGrpYglbeD6QNJQVJkhk%2Fimg.png)*http://www.tcpschool.com/java/java_api_object*

## Dynamic Dispatch

> 컴파일타임에는 알 수 없는 메서드의 의존성을 런타임에 늦게 바인딩

```java
public class Test {
    public static void main(String[] arg) {
        Dispatchable dispatch = new Dispatch();
        System.out.println(dispatch.method());
    }
}

class Dispatch implements Dispatchable {
    public String method(){
        return "hello dispatch";
    }
}

interface Dispatchable{
    String method();
}
```

- 이 경우 컴파일러는 타입에 대한 정보를 알고있으므로 런타임시에 호출 객체를 확인해 해당 객체의 메서드를 호출
- 런타임시에 호출 객체를 알 수 있으므로 바이트코드에도 어떤 객체의 메서드를 호출해야하는지 드러나지 않는다.
- 자바는 묵시적으로 항상 호출 객체를 인자로 보내게된다. 이 this를 통해 메서드 내부에서 호출 객체 참조 가능
- 이는 클래스를 상속했을 때에도 동일하게 적용 된다.

## Double Dispatch
  
```java
public class Test {
    public static void main(String[] arg) {
        List<SmartPhone> phoneList = Arrays.asList(new Iphone(), new Gallaxy());
        Game game = new Game();
        phoneList.forEach(game::play);
    }
}

interface SmartPhone{
}

class Iphone implements SmartPhone{

}

class Gallaxy implements SmartPhone{

}

class Game {
    public void play(SmartPhone phone) {
        System.out.println("game play [" +phone.getClass().getSimpleName()+ "]");
    }
}
```

```java
class Game {
    public void play(SmartPhone phone) {
        if(phone instanceof Iphone) {
            System.out.println("iphone play [" + phone.getClass().getSimpleName() + "]");
        }

        if(phone instanceof Gallaxy) {
            System.out.println("gallaxy play [" + phone.getClass().getSimpleName() + "]");
        }
    }
}
```

- 만약 SmartPhone의 구현체로 Optimus가 추가된다면 Game 클래스까지 변경이 발생

```java
class Game {
    public void play(Iphone phone) {
        System.out.println("iphone play [" + phone.getClass().getSimpleName() + "]");
    }

    public void play(Gallaxy phone) {
        System.out.println("gallaxy play [" + phone.getClass().getSimpleName() + "]");
    }
}
```

```java
public static void main(String[] arg) {
    List<SmartPhone> phoneList = Arrays.asList(new Iphone(), new Gallaxy());
    Game game = new Game();
    phoneList.forEach(game::play); // 컴파일 에러 발생
}
```

- 자바는 묵시적 형변환 지원X
  - 2개의 요소가 서로 타입이 다르므로 명시적 형변환을 하려면 반복문을 사용할 수 없다. 
  - 제네릭같은 방법을 이용한다고 하더라도 구현체마다 다른 행동을 해야하므로 하나의 메서드로 모으는건 의미가 없다.


```
런타임시에 어떤 객체가 들어오는지를 확인해서 서로 다른 메서드를 호출해주는 동적 디스패치가 존재한다면 이를 인자에도 적용?
```

- 지원 않됨

> 이런 이유로 자바를 싱글 디스패치(Single Dispatch) 언어라고 한다.

**해결책**

```java
interface SmartPhone{
    void game(Game game);
}

class Iphone implements SmartPhone{
    @Override
    public void game(Game game) {
        System.out.println("iphone play [" + this.getClass().getSimpleName() + "]");
    }
}

class Gallaxy implements SmartPhone{
    @Override
    public void game(Game game) {
        System.out.println("gallaxy play [" + this.getClass().getSimpleName() + "]");
    }
}

class Game {
    public void play(SmartPhone phone) {
        phone.game(this);
    }
}
```
1. play() 메서드를 찾기위한 정적 디스패치가 발생
2. game()메서드를 호출하는 객체를 찾기위한 동적 디스패치가 발생

- 동적 디스패치가 1번 더 발생하면서 생기는 더블 디스패치(Double Dispatch)