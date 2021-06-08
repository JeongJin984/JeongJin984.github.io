---
title: Network-AMQP
author: Jiny
date: 2021-06-08 10:30:00 +0800
categories: [Network, Protocal]
tags: [network, protocal, amqp]
toc: false
---
 
# Network-AMQP
___

## 💿 **AMQP 통신**

> Advanced Message Queing Protocal의 약자로 흔히 알고있은 MQ의 오픈소스에 기반한 표준 프로토콜

- 가장 많이 쓰는 건 erlang과 자바에 기반을 둔 RabbitMQ
___

## 💿 **등장 배경**

- 상용화된 MQ 제품들
  - 대부분 플랫폼 종속적: 서로 다른 서비스 간에 메시지를 교환하기 위해서는 메세지 포멧 컨버전을 통한 메시지 브릿지를 사용하거나 시스템 자체를 통합할 필요가 있었음
  - MQ는 금융 쪽에서 많이 사용됨
  - 이러한 약점을 사용 보완하는 것이 AMQP
- 서로 다른 시스템 간에 최대한 효율적으로 메시지를 교환하기 위한 MQ 프로토콜
- 벤더에 종속되는 것을 방지하기 위해 아래와 같은 조건을 충족
  - 모든 broker, client들은 똑같은 방식으로 동작할 것
  - 네트웍상으로 전송되는 명령어들의 표준화
  - 프로그램 언어 중립적
___

## 💿 **AMQP Routing model**

![image](https://blog.dudaji.com/assets/rabbitmq/MessageFlow.png)

### **Exchange**

> publisher로 부터 수신한 메시지를 적절한 큐 또는 다른 exchange로 분배하는 라우터 기능을 한다.

- Exchange는 수신한 메시지를 분배하기위해 Exchange Type이라는 라우팅 알고리즘을 사용
  - 브로거는 여러개의 Exchange Type을 가집니다.
- Exchange Type은 받은 메시지를 어떤  방법으로 라우팅할지 결정하는것
  - Binding은 이러한 방법으로 결정된 메시지를 어느 Queue에 전달할지 결정하는 Routing Table

**Exchange Type**

| 타입  |  설명             | 특징 |
|:------|:-----------------|--------:|
| Direct  | Routing key가 정확히 일치하는 Queue에 메시지 전송 | Unicast |
| Topic   | Routing key 패턴이 일치하는 Queue에 메시지 전송   | Multicast |
| Headers | [key:value]로 이루어진 header 값을 기준으로 일치하는 queue에 전송 | Multicast |
| Fanout  | 해당 Exchange에 등록된 모든 Queue에 메세지 전송 | BroadCast |

**Direct Exchange**

![image](https://lostechies.com/content/derekgreer/uploads/2012/03/DirectExchange_thumb1.png)

메시지의 라우팅 키를 큐에 1:1으로 매칭시키는 방법

일반적으로 큐의 이름을 바인딩고자 하는 라우팅 키와 동일하게 작성하는 방법이 있다.

**Topic Exchange**

> 와일드카드를 이용해서 메시지를 큐에 매칭시키는 방법

![image](https://lostechies.com/content/derekgreer/uploads/2012/03/TopicExchange_thumb2.png)

메시지의 라우팅 키를 큐에 1:N으로 매칭시키는 방법

**Headers Exchange**

![image](https://lostechies.com/content/derekgreer/uploads/2012/03/HeadersExchange_thumb2.png)

key-valie로 정의된 헤더에 의해 라우팅을 결정한다.

x-match라는 특별한 argument로 헤더를 어떤식으로 해석하고 바인딩할지를 결정하는데, 
- x-match가 all이면 바인딩 조건을 모두 충족시켜야 한다는 것이고(AND), 
- any이면 하나만 충족시키면 된다는 것이다.

**Fanout Exchange**

![image](https://lostechies.com/content/derekgreer/uploads/2012/03/FanoutExchange_thumb2.png)

모든 메세지를 모든 큐로 라우팅하는 유형이다.

### **Queue**

> 메모리나 디스크에 메시지를 저장하고 그것을 Consumer에게 전달하는 역할

- 큐는 스스로가 관심있는 메시지 타입을 지정한 Binding을 통해 exchange에 말그대로 binding 된다.

### **Binding**

> exchange와 Queue와의 관계를 정의한 일종의 라우팅 테이블

- 같은 큐가 여러개의 exchange에 bind 될 수도 있고, 하나의 exchange에 여러개의 큐가 binding 될 수도 있다.

### **Routing Key**

> Publisher에서 송신한 메시지 헤더에 포함되는 것으로 일종의 가상 주소

- Exchange는 이것을 이용해서 어떤 큐로 메시지를 라우팅할건지 결정할 수 도 있다.

