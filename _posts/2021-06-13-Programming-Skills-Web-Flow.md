---
title: Programming Skills 2
author: Jiny
date: 2021-06-13 14:30:00 +0800
categories: [Backend, Basics]
tags: [web, backend]
toc: false
---
 
# Programming Skills 2

___

## 💿 **Serverless**

> Baas 와 Faas에 의존하는 아키텍처를 Serverless()

**Bass:** 자주 필요한 사용자 관리 및 접속 제어, 푸시 알림, 데이터 저장, 누리 소통망 서비스(SNS), 위치 서비스 등의 백엔드 기능을 구현하기 위해 코드를 직접 개발해야 한다. 그러나 서비스형 백엔드(BaaS)를 이용하면, 직접 코드를 개발하지 않고 앱을 클라우드와 연동시켜 BaaS에서 제공하는 응용 프로그램 인터페이스(API: Application Program Interface)를 호출하여 사용할 수 있다.

**Faas:** 사전 작성된 서비스 라이브러리에 의존하지 않고 사용자 정의 애플리케이션을 생성하는 개발자에게 더 많은 제어 권한을 제공,
- Amazon Web Services의 AWS Lambda, Microsoft Azure의 Azure Functions, Google Cloud의 여러 오퍼링, IBM Cloud의 IBM Cloud Functions 등

Serverless 와 같이 인프라 환경이 고도로 자동화/추상화된 환경에서는 이제 전통적인 JVM, 커널 파라미터 등은 필수 지식이 아니라고 생각할 수도 있습니다.
- 하지만 내부적으로 Baas, Faas를 만들기도 하고
- 인프라 구조를 잘 이해하면 추상화된 서비스도 잘 쓸 수 있게 될 여지도 있음

___

## 💿 **Client 와 백엔드**

> 예전에는 JSP나 HTML을 서버사이드에서 생성하는 기술이 주로 쓰였습니다. 화면 전체를 다시 그릴 필요가 없는 요청은 Ajax로 처리했지만 서버 렌더링에 보조적으로 쓰이는 경향이 강했음

- 현재는 React나 Vue 같은 프레임워크가 널리 쓰이면서 클라이언트 사이드에서 HTML을 생성하는 비중이 높아지고 있음
- 클라이언트 사이드 렌더링을 하는 구조는 서버 개발자가 HTTP API 개발에만 전념할 수 있다는 장점이 있음

___

## 💿 **Client 와 백엔드**

> HTTP API의 설계의 많은 부분은 매번 프로젝트마다 고민해야할 점이 많음

- 단순 CRUD API는 각각 POST/GET/PUT/DELETE의 HTTP 메서드로 연결시키더라도 이를 벗어난 기능들은 어떻게 설계해야할지 명확하지 않은 경우가 많음
  - 예를 들어 페이징 처리, 복잡한 검색조건이 있을때의 파라미터 표현방식
- Client 사이드에서 UI 렌더링을 하는 경우가 늘어나면서 HTTP API를 다양하고 정교하게 사용하고자 하는 필요성이 늘었음
  - 이런 상황 속에서 FaceBook사에서는 GraphQL을 내놓았고
  - Spring 에서는 HateOAS를 통해 한단계 REST를 업그레이드 했음

___

## 💿 **시스템을 어떻게 자를것인가**

> 시스템을 만들 때 많은 고민들은 결국 '구성요소 간의 역할과 책임을 어떻게 나눌 것인가'로 표현

- 최근 MSA라는 구조가 각광 받으면서 서버에 배포 가능한 모듈의 단위를 이전보다 작게 가져가는 경향이 나타나고 있음
  - 예전에는 MSA와 같은 구조로 서비스를 만드는 것이 비용이 더 컸었습니다.
  - 요즘에는 인프라 시스템, 모니터링, 프레임워크의 발전으로 비용이 내려갔음

___

## 💿 **인프라 기술 활용**

- Docker + Kubernetes를 이용한 빌드 서버 가상화 사례
- DEVIEW 2015의 Docker Orchestration 발표를 참고하실 수 있습니다.

> 네이버 내부에 정확히 'IT operation팀'이라는 용어는 쓰이고 있지는 않습니다. Nginx, Tomcat, JVM과 같은 솔루션의 설치와 설정 변경은 개발팀에서 직접하고 있습니다. 

- 운영서버의 VM 생성, OS패치등의 작업, 인프라 구성 등에 대한 컨설팅을 서비스마다 지정된 SE분들이 해주심
- 개발DB의 설치와 스키마 변경은 개발팀에서 자율적으로 하고, 운영 DB에 대한 중요한 작업은 전담 DBA에게 요청