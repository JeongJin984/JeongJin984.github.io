---
title: Java Thread
author: Jiny
date: 2021-01-22 22:40:00 +0800
categories: [Java, Jasic]
tags: [java, basic]
toc: false
---

# Java Thread
___

## Thread 생성자

|함수|설명|
|----|---|
|Thread()|스레드 이름|
|Thread(Runnable r)|인터페이스 객체|
|Thread(Runnable r, String s)|인터페이스 객체와 스레드 이름|

## Thread Method

|함수|설명|
|----|---|
|static void sleep(long msec)<br/> throws Interrupted Exception|msec에 지정된 밀리초 동안 대기|
|String getName()|스레드의 이름을 s로 설정|
|void setName(String s)|스레드의 이름을 s로 설정|
|void start()|스레드를 시작 run() 메소드 호출|
|int getPriority()|스레드의 우선 순위를 반환|
|void setpriority(int p)|스레드의 우선순위를 p값으로|
|boolean isAlive()|스레드가 시작되었고 아직 끝나지 않았으면 <br/>true 끝났으면 false 반환|
|void join() <br/> throws InterruptedException| 스레드가 끝날 때 까지 대기 |
|void run()| 스레드가 실행할 부분 기술<br/> (오버라이딩 사용)|
|void suspend()|스레드가 일시정지 resume()에 <br/>의해 다시시작 할 수 있다. |
|void resume()|일시 정지된 스레드를 다시 시작.|
|void yield()|다른 스레드에게 실행 상태를 양보하고 <br/>자신은 준비 상태로|

## Thread 생성
 
### extend Thread
```java
public class Test extends Thread {
    int seq;
    public Test(int seq) {
        this.seq = seq;
    }
    public void run() {
        System.out.println(this.seq+" thread start.");
        try {
            Thread.sleep(1000);
        }catch(Exception e) {

        }
        System.out.println(this.seq+" thread end.");
    }

    public static void main(String[] args) {
        for(int i=0; i<10; i++) {
            Thread t = new Test(i);
            t.start();
        }
        System.out.println("main end.");
    }
}
```

**output**
```
0 thread start.
4 thread start.
6 thread start.
2 thread start.
main end.
3 thread start.
...
1 thread end.
5 thread end.
```
- 특징
  - test.start() 실행 시 test객체의 run 메소드가 수행
  - 쓰레드가 종료되기 전에 main 메소드가 종료
 
### Join

> 쓰레드가 종료될 때까지 기다리게 하는 메서드

```java
public static void main(String[] args) {
    ArrayList<Thread> threads = new ArrayList<Thread>();
    for(int i=0; i<10; i++) {
        Thread t = new Test(i);
        t.start();
        threads.add(t);
    }

    for(int i=0; i<threads.size(); i++) {
        Thread t = threads.get(i);
        try {
            t.join();
        }catch(Exception e) {
        }
    }
    System.out.println("main end.");
}
```

**output**

```
0 thread start.
5 thread start.
2 thread start.
6 thread start.
...
main end.
```

- 특징
  - 모든 쓰레드가 종료된 후에 main 메소드를 종료
  
### Runnable

> 흔히 사용되는 방법

```java
public class Test implements Runnable {
    int seq;
    public Test(int seq) {
        this.seq = seq;
    }
    public void run() {
        System.out.println(this.seq+" thread start.");
        try {
            Thread.sleep(1000);
        }catch(Exception e) {
        }
        System.out.println(this.seq+" thread end.");
    }

    public static void main(String[] args) {
        ArrayList<Thread> threads = new ArrayList<Thread>();
        for(int i=0; i<10; i++) {
            Thread t = new Thread(new Test(i));
            t.start();
            threads.add(t);
        }

        for(int i=0; i<threads.size(); i++) {
            Thread t = threads.get(i);
            try {
                t.join();
            }catch(Exception e) {
            }
        }
        System.out.println("main end.");
    }
}
```
- 특징
  - Thread t = new Thread(new Test(i));







