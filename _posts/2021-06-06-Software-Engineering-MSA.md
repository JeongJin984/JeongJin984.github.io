---
title: Software Engineering MSA
author: Jiny
date: 2021-06-06 10:30:00 +0800
categories: [Architecture, MSA]
tags: [architecture, msa]
toc: false
---
 
# MSA(Micro Service Architecture)
___

## 💿 **MSA**

> Monolith: '한 덩어리'에 해당하는 구조로 이루어져 있다. 모든 기능을 하나의 어플리케이션에서 비즈니스 로직을 구성해 운영한다. 

- 개발을 하거나 설정에 있어서 간단하기에 작은 사이즈의 프로젝트에서는 유리하지만, 
- 시스템이 점점 확장되거나 큰 프로젝트에서는 단점들이 존재한다.
  - 빌드/테스트 시간의 증가: 하나를 수정해도 시스템 전체를 빌드해야함, 유지 보수가 힘들다.
  - 작은 문제가 시스템 전체에 문제를 일으킴: 서비스 하나의 트래픽 문제로 서버가 다운되면 모든 서비스가 이용 불가능해 진다.
  - 확장성에 불리: 하나의 서비스를 확장하기 위해 전체 프로젝트를 확장해야 한다.

> MSA: 단일 프로그램을 각 컴포넌트 별로 나누어 작은 서비스의 조합으로 구축하는 방법

![image](https://camo.githubusercontent.com/6fd425b074f303b86584946ba4c5ec8d9aa8169ca55d6b7703d9f990d29ecfda/68747470733a2f2f7777772e7265646861742e636f6d2f636d732f6d616e616765642d66696c65732f6d6f6e6f6c69746869632d76732d6d6963726f73657276696365732e706e67)

![image](https://camo.githubusercontent.com/c2fe9882737d9322fc3718681d71b704e45b389f3fb4f7e7829cfb23f6c6200d/68747470733a2f2f6d69726f2e6d656469756d2e636f6d2f6d61782f313235302f312a565f6d7656306d626663426f43636972446773765a772e706e67)

MSA에서는 각 컴포넌트는 API를 통해 다른 서비스와 통신하는데 모든 서비스는 각각 독립적인 서버로 운영하고 배포하기 때문에 서로 의존성이 없다. 따라서 하나의 서비스에 문제가 생겨도 다른 서비스에는 영향을 끼치지 않으며 서비스 별로 부분적인 확장이 가능하다.

___

## 💿 **MSA 표준 구성요소**

![image](https://media.vlpt.us/post-images/tedigom/b6bae160-fb10-11e9-9ef4-395edd3ef4d0/%EA%B0%80%ED%8A%B8%EB%84%88MSAComponent.png)

MSA는 크게 Inner Architecture 와 Outer Architecture로 구분할 수 있습니다. 위 그림에서 남식 부분은 Inner 이고 회식 부분은 Outer 입니다.

### **Inner Architecture**

- 고려 사항
  - 서비스를 어떻게 정의(나눌)할 것인가.
    - 서비스를 정의하기 위해서 고려해야 할 사항은 비즈니스, 서비스 간 종속성, 배포 용이성, 장애 대응, 운영 효율성
  - DB Access 구조를 어떻게 설계할 것인다.
    - MSA가 사용하는 데이터는 일반적으로 일관된 API를 통해서 접근합니다. 또한 각 서비스에는 자체의 DB를 사용하는데 일부 트랜잭션은 여러 microService에 걸쳐 있기 떄문에 각 서비스에 연결된 DB의 정합성을 보장할 수 있는 방안이 필요
  - api 설계
  - 논리적인 컴포넌트들의 Layer는 어떻게 설계할 것인가.
    - 도커 layer에 관한 이야기 인 것 같다.

### **Inner Architecture**

![image](https://media.vlpt.us/post-images/tedigom/052a8e50-fbdd-11e9-9256-3bbc0d52572a/Inner.PNG)

위와 같이 Gartner에서는 MSA의 Outer architecture을 총 6개의 영역으로 분류하고 있습니다.

1. External Gateway
2. Service Mash
3. Container Management
4. Backing Services
5. Telemetry
6. CI/CD Automation

___

## 💿 **Inner Architecture**

### **External Gateway**

![image](https://docs.microsoft.com/en-us/azure/architecture/microservices/images/gateway.png)

> 전체 서비스 외부로부터 들어오는 접근을 내부 구조를 드러내지 않고 처리하기 위한 요소입니다.

- 사용자 인증(Consumer Identity Provider)과 권한 정책 관리(policy management)를 수행하며, API Gateway가 가장 핵심적인 역할을 담당합니다.

### **Service Mesh**

![image](https://media.vlpt.us/post-images/tedigom/3fbe7db0-0925-11ea-aa94-f3699ad0167a/service-mesh-generic-topology.png)


>  마이크로 서비스 간의 통신(네트워크)을 담당하는 요소

- 마이크로 서비스 구성 요소간 상호 통신을 위해서는 Service Discovery, 서비스 라우팅, Failure recovery, load balancing(트래픽 관리), 보안 등의 문제를 처리할 수 있는 메커니즘이 필요하다
- Service Mesh는 통신 및 네트워크 기능을 비즈니스 로직과 분리한 네트워크 통신 인프라. 모든 서비스의 인프라 레이어로서 서비스들 간의 통신을 처리하며, 위의 언급된 많은 기능들을 포함

**API Gateway와의 차이점**

- 적용되는 위치
  -  API Gateway는 마이크로서비스 그룹의 외부 경계에 위치하여 역할을 수행하지만, ServiceMesh는 경계 내부에서 그 역할을 수행합니다.
- 아키텍쳐 형태
  - API Gateway가 중앙집중형 아키텍쳐여서 SPOF(Single Point of Failure)을 생성한다면, 
  - Service Mesh는 분산형 아키텍쳐를 취하기 때문에 SPOF를 생성하지 않고 확장이 용이
- 패턴
  - API Gateway는 일반적으로 Gateway proxy pattern을 사용해서 수행됩니다. Consumer(호출자)은 구현 내용을 알 필요 없이 Gateway를 호출하는 방법만 알면 Gateway가 알아서 수행해주는 방식
  - Sidecar proxy pattern을 사용합니다. Consumer(호출자)의 코드에는 Provider(공급자)의 주소를 찾는방법, failover와 관련된 코드 등의 내용이 들어가게 됩니다. 다만, 호출자의 코드는 어플리케이션 코드(비즈니스 로직)에 내장되는 것이 아니라 sidecar 형태로 별개로 관리

최근 MSA에서 API Gateway는 노출되는 부분에 위치하여 내부 서비스를 보호 및 제어하는 역할을 하고, 

Service Mesh는 내부 서비스에 위치하여 서비스를 관리하는 구조로 많이 사용됨

**유형**

1. PaaS (Platform as a Service)의 일부로 서비스 코드에 포함되는 유형
   - Microsoft Azure Service fabric, lagom, SENECA 등이 이 유형에 해당되며, 프레임워크 기반의 프로그래밍 모델이기 때문에, 서비스메쉬를 구현하는데에 특화된 코드가 필요합니다. ( Mesh-native Code )

2. 라이브러리로 구현되어 API 호출을 통해 Service mesh에 결합되는 유형
   - Spring Cloud, Netflix OSS(Ribbon/Hystrix/Eureka/Archaius), finagle 등이 이 유형에 해당되며, 프레임워크 라이브러리를 사용하는 형태입니다. 이중 Netfilix의 Prana는 sidecar 형태로 동작합니다. 서비스 메시를 이해하고 코드를 작성해야합니다. (Mesh Aware Code)

3. Side car proxy를 이용하여 Service mesh를 마이크로서비스에 주입하는 유형
   - Istio/Envoy, Consul, Linkerd 등이 이 유형에 해당되며, sidecar proxy 형태로 동작됩니다. 따라서 서비스메시와 무관하게 코드를 작성할 수 있습니다.

### **Container Management**

![image](https://media.vlpt.us/post-images/tedigom/fc7292b0-fd32-11e9-ba49-23d182ee1325/container-management-diagram.png)

컨테이너 기반 어플리케이션 운영은 유연성과 자율성을 가지며, 개발자가 손쉽게 접근 및 운영할 수 있는 인프라 관리 기술이기 때문에 MSA에 적합하다고 평가받고 있음

kubernetes에 관한 것은 따로 Post에서 다루겟음

### **Backing Service**

![image](https://media.vlpt.us/post-images/tedigom/1f0216d0-fd37-11e9-ae72-731624d4042b/messagequeue.png)

MSA 에서 특징적인 Backing Service 중 하나인 Message queue

MSA에서는 메세지의 송신자와 수신자가 직접 통신하지 않고 Message Queue를 활용하여 비동기적으로 통신하는 것을 지향함

Message Queue를 사용하지 않는 강한 결합 구조의 경우, 여러 서비스를 걸치는 실시간 트랜잭션을 처리할 때, 하나의 서비스가 죽어버린다면 트랜잭션이 끊어지기 때문에 해당 서비스 요청을 보존할 수 없고 큰 에러가 발생

MSA에서 데이터 변경이나, 보상 트랜잭션과 관련된 처리는 Message Queue를 활용한 비동기 처리가 효율적입니다.

이러한 Message Queue에는 카프카, RabbitMq 등이 있는데 이는 추후에 다시 다루겠음

### **Telemetry**

> 실시간 원격 성능 측정

MSA에서 상당수의 마이크로 서비스가 분산 환경에서 운영되기 때문에 서비스들의 상태를 모니터링하고 이슈에 대응하는것은 매우 고되고 시간이 오래 걸리는 일

 Telemetry는 서비스들을 모니터링하고, 서비스별로 발생하는 이슈들에 대응할 수 있도록 환경을 구성하는 역할

 Grafana + Prometheus를 통해 성능 측정 및 로그 저장이 가능 이도 추후에 다루겠음