---
title: Thread Process
author: Jiny
date: 2021-01-22 22:40:00 +0800
categories: [Java, Jasic]
tags: [java, basic]
toc: false
---

# Thread Process
___

## Definition

- **프로그램**: 파일이 저장 장치에 저장되어 있지만 메모리에는 올라가 있지 않은 정적인 상태를 말한다.
- **프로세스**: 운영체제로부터 자원을 할당받은 작업의 단위.
- **스레드**: 프로세스가 할당받은 자원을 이용하는 실행 흐름의 단위.


## Thread & Process

![image](https://media.vlpt.us/images/raejoonee/post/6f274681-dfa7-45eb-9121-2cc9f4b972a5/102.png)

Thread는 Process와 다르게 Thread 간 Memory를 공유하며 작동
- 즉 Thread는 Process의 자원을 공유하면서 Process 실행 흐름의 일부가 된다.

## Thread

![image](https://media.vlpt.us/images/raejoonee/post/b7939911-f3e8-48ac-abb8-63d8a17d0444/103.png)

- 운영체제는 프로세스마다 각각 독립된 메모리 영역을, Code/Data/Stack/Heap의 형식으로 할당
- 이와 다르게 Thread는 Memory를 서로 공유 할 수 있음

![image](https://media.vlpt.us/images/raejoonee/post/b91490ed-c67b-407d-8fea-a8d6fdb22559/104.png)

- Process 강제 종료: 다른 프로세스 는 공유하고 있는 파일을 손상시키는 경우가 아니라면 아무런 영향을 받지 않음.
- Thread 강제 종료: Thread 하나에서 오류가 발생한다면 같은 Process 내의 다른 Thread 모두가 강제로 종료

> 앞서 언급한 대로 Thread의 경우 동기화, 간섭이 가장 큰 단점으로 작용

## Process

**프로세스도 공유가 됨**

- IPC(Inter-Process Communication)을 사용한다.
- LPC(Local inter-Process Communication)을 사용한다.
- 별도로 공유 메모리를 만들어서 정보를 주고받도록 설정해주면 된다.

> CPU 레지스터 교체뿐만이 아니라 RAM과 CPU 사이의 캐시 메모리까지 초기화되기 때문에 앞서 말했듯 자원 부담이 크다.