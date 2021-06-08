---
title: Data Engineering Kafka
author: Jiny
date: 2021-06-08 14:30:00 +0800
categories: [DataEngineering, Kafka]
tags: [kafka, data]
toc: false
---
 
# Data Engineering Kafka
___

![image](https://docs.confluent.io/5.5.1/_images/kafka-apis.png)

> Pub-Sub 모델의 메시지 큐

- 분산환경에 특화

## 💿 **구성**

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile30.uf.tistory.com%2Fimage%2F99725F4F5C04E42B35922A)

### **Event**

> Producer와 Consumer가 데이터를 주고 받는 단위

### **Producer**

> 데이터를 발생시키고 카프카 클러스터(Kafka Cluster)에 적재하는 프로세스

### **Kafka Cluster**

> 카프카 서버로 이루어진 클러스터

- **브로커**: 카프카 서버
- **주키퍼(zookeeper)**: 분산 코디네이션 시스템. 카프카 브로커를 하나의 클러스터로 코디네이팅하는 역할을 하며 리더 파티션을 발탁하는 방식도 주키퍼가 제공하는 제공을 이용
- **토픽**: 카프카 클러스터에서 데이터를 관리하는 기준. 카프카 클러스터에서 여러개 만들 수 있으며 하나의 토픽은 1개 이상의 파티션으로 구성되어 있습니다. 즉 어떤 데이터를 관리하는 하나의 그룹
- **파티션**: 각 토픽 당 데이터를 분산 처리하는 단위. 카프카에서는 토픽 안에 파티션을 나누어 그 수대로 데이터를 분산 처리. 카프카 옵션에서 지정한 replica의 수 만큼 파티션이 브로커에 복제됨
- **리더, 팔로워**: 카프카에서는 각 파티션당 복제된 파티션 중에서 하나의 리더가 선출 이 리더는 모든 릭기 쓰기 연산을 담당. 리더를 제외한 나머지는 팔로워가 되고 이 팔로워 들은 단순히 리더의 데이터를 복사하는 역할(리더가 고장나면 리더로 변한다.)

### **Consumer Group**

> 컨슈머의 집합을 구성하는 단위. 카프카에서는 컨슈머 그룹으로 데이터를 처리하며 컨슈머 그룹 안의 컨슈머 수만큼 파티션의 데이터를 분산 처리

위 그림에서는 Producer 가 데이터를 적재하고 있으며 데이터를 Consumer Group A 와 B가 각각 자신이 처리해야될 Topic Foo 와 Bar를 가져오고 있음

파티션은 늘릴수는 있지만 **줄일수는 없다.**

자신이 가져와야 하는 토픽 안의 파티션의 데이터를 Pull하게 되고 각각 컨슈머 그룹 안의 컨슈머 들이 파티션이 나뉜 만큼 데이터를 처리

___

## 💿 **카프카 파티션 읽기, 쓰기**

> 카프카에서의 쓰기, 읽기 연산은 카프카 클러스터 내의 리더 파티션들에게만 적용

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile26.uf.tistory.com%2Fimage%2F9911C9475C04EAFA0592D8)

데이터를 순차적으로 디스크에 저장(큐 형식)

write 할 때 파티션들은 각각의 데이터들의 순차적인 집합인 오프셋을 구성

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile21.uf.tistory.com%2Fimage%2F99BD59475C04E2D8322293)

컨슈머그룹의 각 컨슈머들은 파티션의 오프셋을 기준으로 데이터를 순차적으로 처리

컨슈머들은 컨슈머 그룹으로 나뉘어서 데이터를 분산 처리하게 되고 같은 컨슈머 그룹내에 있는 컨슈머 끼리 같은 파티션의 데이터를 처리할 수 없음

이 데이터들은 설정값에 따라 데이터를 디스크에 보관하게 됩니다. (2.x 기준 default 7일)

