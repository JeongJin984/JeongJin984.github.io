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

## Thread LifeCycle

![image](https://miro.medium.com/max/700/1*Dfl8EQlWdIebwAh9UinLMA.jpeg)

- New: Thread.start() 하면 실행됨
- Runnable: start하면 runnable로 바뀜
  - Control이 Thread Scheduler로 가서 실행을 기다림
- Running: Scheduler가 실행을 시킨 상태
- Blocked
  - Waiting for I/O resources
  - Waiting for a monitor lock
- Waiting: 다른 Thread가 특정 action을 perform하길 기다리는 상태
  - Object.wait with no timeout
  - Thread.join with no timeout
  - 예를 들어 Object.wait()를 특정 object에 호출한 후 다른 쓰레드가 Object.notify or Object.notifyAll()를 그 Object에 호출하기를 기다리는 상태
  - waiting이 끝나면 Runnable로 돌아감
- Timed_Waiting: 밑의 함수를 호출 해서 waiting 상태
  - Thread.sleep
  - Object.wait with timeout
  - Thread.join with timeout
- Terminated: run()이 끝나면 자원을 반납하고 종료

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

## Thread 우선순위

> 우선순위라는 속성(멤버변수)의 값에 따라 쓰레드가 얻는 실행시간이 달라짐

- 쓰레드가 수행하는 작업의 중요도에 따라 쓰레드의 우선순위를 서로 다르게 지정하여 특정 쓰레드가 더 많은 작업시간을 갖도록 할 수 있다

```java
class ThreadPriority {

	public static void main(String args[]) {

		A th1 = new A();
		B th2 = new B();

		th1.setPriority(4); // defalut 우선순위 5
		th2.setPriority(7);

		System.out.println("Priority of th1(-): " + th1.getPriority());
		System.out.println("Priority of th2(|): " + th2.getPriority());

		th1.start();
		th2.start();

	}

}

class A extends Thread {

	public void run() {
		for(int i=0; i < 300; i++) {
			System.out.print("-");
			for(int x=0; x < 10000000; x++);
		}
	}
}

class B extends Thread {

	public void run() {
		for(int i=0; i < 300; i++) {
			System.out.print("|");
			for(int x=0; x < 10000000; x++);
		}
	}	
}
```

우선순위가 높은 th2의 실행시간이 th1에 비해 상당히 늘어난다.

## Main 쓰레드

> 메인이 수행하는 스레드 public static void main(String[] args)

![image](https://2.bp.blogspot.com/-LR5DRGuoL1g/Wa-KoLJVdvI/AAAAAAAABTQ/8Fl2OhPww6MtTkxxqpNCYmUWv1-Ftan0QCLcBGAs/s1600/main-thread-in-java.jpeg)

- The main thread is created automatically when our program is started
  
### Main Thread Controll

```java
public class Test extends Thread {
	public static void main(String[] args) {

	// getting reference to Main thread
	Thread t = Thread.currentThread();
  
	// getting name of Main thread
	System.out.println("Current thread: " + t.getName());
  
	// changing the name of Main thread
	t.setName("Geeks");
	System.out.println("After name change: " + t.getName());
  
	// getting priority of Main thread
	System.out.println("Main thread priority: "+ t.getPriority());
  
	// setting priority of Main thread to MAX(10)
	t.setPriority(MAX_PRIORITY);
  
	System.out.println("Main thread new priority: "+ t.getPriority());
  
	for (int i = 0; i < 2; i++) {
		System.out.println("Main thread");
	}
  
	// Main thread creating a child thread
	ChildThread ct = new ChildThread();

	// getting priority of child thread
	// which will be inherited from Main thread
	// as it is created by Main thread
	System.out.println("Child thread priority: "+ ct.getPriority());

	// setting priority of Main thread to MIN(1)
	ct.setPriority(MIN_PRIORITY);

	System.out.println("Child thread new priority: "+ ct.getPriority());

	// starting child thread
	ct.start();
	}
}

// Child Thread class
class ChildThread extends Thread {
	@Override
	public void run() {
		for (int i = 0; i < 2; i++){
			System.out.println("Child thread");
		}
	}
}
```

**output**

```
Current thread: main
After name change: Geeks
Main thread priority: 5
Main thread new priority: 10
Main thread
Main thread
Child thread priority: 10
Child thread new priority: 1
Child thread
Child thread
```