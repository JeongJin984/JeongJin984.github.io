---
title: SIY Spring Cloud Eureka
author: Jiny
date: 2021-06-06 16:30:00 +0800
categories: [Spring, Cloud]
tags: [spring, cloud, msa, eureka, siy]
toc: false
---
 
# Eureka
___

## 💿 **Eureka**

유레카는 클라이언트와 서버로 나뉘어저 있다.

- 유레카 서버
  - 모든 마이크로 서비스가 자신의 가용성을 등록하는 레지스트리
  - 등록되는 정보는 서비스 ID 와 URL이 포함되는데 유레카 클라이언트를 통해서 이 정보를 서버에 등록
  - 등록된 정보를 통해서 서비스의 동적 발견이 가능
- 유레카 클라이언트
  - 등록된 마이크로 서비스를 호출해서 사용할 유레카 클라이언트를 발견(gateway)
  - 유레카 서버는 동시에 유레카 클라이언트이기도 해서 다른 유레카 서버와 상태를 동기화 함
  - 여러대의 유레카 서버를 운영시 고가용성

![image](https://media.vlpt.us/post-images/tedigom/b6bae160-fb10-11e9-9ef4-395edd3ef4d0/%EA%B0%80%ED%8A%B8%EB%84%88MSAComponent.png)
_Service Discovery: Eureka Server / Service Router: Api Gateway_

___

## 💿 **Eureka 구축**

### **Eureka Server**

**주요 Component**

- @EnableEurekaServer(main 클래스): 에 붙이며 유레카 서버를 쓰겠다는 뜻
  - 여기 서버의 index를 호출하면 현재 가용 상태의 서버를 볼 수 있음
- register-with-eureka(yml)
  - 레지스트리에 자신을 등록할지 여부
  - 클러스터링 모드의 유레카 서버 구성은 서로 peering 구성이 가능. (유레카 서버 설정에 정의된 peering 노드를 찾아서 레지스트리 정보의 sync를 맞춤)
- fetch-registry(yml)
  - 레지스트리에 있는 정보를 가져올 지에 대한 여부
- wait-time-in-ms-when-sync-empty
  - registry를 갱신할 수 없을때 재시도를 기다리는 시간
  - 테스트 시 짧은 시간으로 등록해놓으면 유레카 서비스의 시작 시간과 등록된 서비스를 보여주는 시간 단축 가능
  - 유레카는 등록된 서비스에서 10초 간격으로 연속 3회의 상태 정보(heartbeat)를 받아야 하므로 등록된 개별 서비스를 보여주는데 30초 소요

**주의 사항**

- 유레카 클라이언트를 1개 구성할 때는 register-with-eureka, fetch-registry 둘다 false
- 