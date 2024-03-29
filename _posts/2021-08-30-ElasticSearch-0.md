---
title: Elasticsearch 0
author: Jiny
date: 2021-08-30 14:30:00 +0800
categories: [ElasticSearch]
tags: [elasticsearch, bigdata, elk]
toc: false
---
 
# Elasticsearch 0

___

## 💿 **개념**

> 아파치 루씬(Lucene) 기반의 오픈소스 실시간 분산 검색 엔진으로 JSON 기반의 비정형 데이터 분산 검색 및 분석을 지원한다.

- 설치와 서버 확장이 매우 편리
- 실시간 검색 서비스 지원, 분산 및 병렬 처리, 그리고 멀티테넌시 기능 제공
- 현재 웹 문서 검색, 소셜 데이터 분석, 쇼핑몰 등에 활용되고 있음
- 빅데이터 분석/처리 MSA 환경의 로그 모니터링등에 활용

___

## 💿 **특징**

### **분산/확장성/병렬처리**

Elasticsearch 구성 시 보통 3개 이상의 노드(Elasticsearch 서버)를 클러스터로 구성하며, 데이터를 샤드(shard)로 저장 시 클러스터 내 다른 호스트에 복사본(replica)을 저장해 놓기 때문에 하나의 노드가 죽거나 샤드가 깨져도 복제되어 있는 다른 샤드를 활용

데이터의 분산과 병렬처리가 되므로 실시간 검색 및 분석을 할 수 있고, 노드를 수평적으로 늘릴 수 있게 설계되어 있기 때문에 더 많은 용량이 필요한 경우 노드를 클러스터에 추가할 수 있다.

### **고가용성**

Elasticsearch는 동작 중에 죽은 노드를 감지하고 삭제하며 사용자의 데이터를 안전하고 접근 가능하도록 유지하기 때문에 동작 중에 일부 노드에 문제가 생기더라도 문제가 없음

### **멀티 테넌시**

클러스터는 여러개의 인덱스(RDBMS의 데이터베이스와 비슷)들을 저장하고 관리하며, 하나의 쿼리가 그룹 쿼리로 여러 인덱스의 데이터를 검색할 수 있다.

### **문서 중심 & 스키마 미존재**

복잡한 현실 세계의 요소들을 구조화된 JSON 문서 형식으로 저장한다. 모든 필드는 기본적으로 인덱싱되며, 모든 인덱스들은 단일 쿼리로 빠르게 검색 및 활용할 수 있다.

또한 NoSQL이나 RDBMS와 같은 스키마 개념이 없으며, 사용자의 데이터가 어떻게 인덱싱 될지를 사용자가 커스터마이징 할 수 있다.

### **플러그인 형태로 구현**

검색엔진을 직접 수행하지 않고, 필요한 기능에 대한 플러그인을 적용하여 기능을 확장할 수 있다. 예를 들면 외부에서 제공하는 형태소 분석기나 REST API를 구현하여 적용할 수 있다.

___

## 💿 **구조**

### **논리적 구조**

![image](https://ssup2.github.io/images/theory_analysis/Elasticsearch_Data_Structure/Elasticsearch_Data_Structure.PNG)

**도큐먼트(Document)**

> 하나의 Doc은 다양한 필드로 구성되어 있으며 이 필드에는 데이터 필드에 해당하는 데이터 타입이 들어감.

- Elasticsearch의 최소단위(RDBMS의 Row와 비슷) Json 오브젝트 하나
- 중첩구조를 지원하기 때문에 Document 내부에 Document가 들어가는 것도 가능함

**타입**

> 여러개의 Document가 모여서 한개의 Type을 이룸. 

- 7.0 부터 Type이 사라졌으며 Index가 그 역할을 대신

**필드**

> Document에 들어가는 데이터 타입으로 RDBMS의 열(Column)과 비슷하다. 하지만 필드는 동적임

- RDBMS는 하나의 열이 하나의 데이터 타입만 가질 수 있지만 Elasticsearch에서는 여러개의 타입을 가질 수 있다.

**매핑**

> 필드와 필드의 속성을 정의하고 색인 방법을 정의

- 매핑 정보에 여러가지 데이터 타입 지정이 가능하지만 필드면 자체는 중복이 불가능

**인덱스**

> RDBMS의 Table + Schema 역할

- RDBMS는 쿼리하나로 여러 Schema를 동시에 조회할 수 없지만, Elasticsearch는 가능함
- Elasticsearch를 클러스터로 구성했을 경우 Index는 여러 노드에 분산 저장/관리 됨
  - 기본설정은 5개의 Priamry shard 와 1개의 Replica shar를 생성함

### **물질적 구조**

**노드**

> Elasticsearch 클러스터에 포함된 단일 서버로 데이터를 저장하고 클러스터의 색화 및 검색 기능에 참여

- RDBMS와는 다르게 여러개의 노드로 구성된 클러스터는 관리하는 데이터가 거져도 노드를 추가시켜서 대용량 처리를 가능하게 해줌
- 구성
  - **마스터 노드(Master Node):** 클러스터 관리 노드. 노드 추가/제거, 인덱스 생성/삭제 등 클러스터의 전반적 관리 담당. 여러개의 마스터 노드를 설정하면 하나의 마스터 노드로 작동됨
  - **데이터 노드(Data Node):** 데이터(Document)가 저장되는 노드. 데이터가 분산 저장 되는 물리적인 공간인 샤드가 배치되는 노드. 색인/검색/통계 등 데이터 작업을 수행
  - **코디네이팅 노드(Coordinating Node):** 사용자의 요청을 받고 Round Robin 방식으로 분산시켜주는 노드. 클러스터에 관련된 것은 마스터 노드로, 데이터는 데이터 노드로 넘김
  - **인제스트 노드(Ingest Node):** 문서 전처리 작업 수행. 인덱스 생성 전 문서 형식 변경을 다양하게 할 수 있음(스크립트로 전처리 파이프라인 구성등)

**샤드(Shard)**

> 인덱스 내부에는 색인된 데이터들이 존재하는데 이 데이터들은 하나로 뭉쳐서 존재하지 않고 물리적 공간에 여러개의 부분들로 나뉘어서 존재함.(이러한 공간을 샤드라 함)

- Primary Shard: 데이터의 원본. Elasticsearch에서 데이터 업데이트 요청을 날리면 반드시 Primary Shard에 요청을 하게 되고 해당 내용은 레플리카에 복제됨.
  - 검색 성능을 위해 클러스터의 샤드 갯수를 조정하는 튜닝을 함
- Replica Shard: 기존 원본 데이터가 무너졌을 때 그 대신 쓰면서 장애 극복 역할을 수행함. 

**세그먼트(Segment)**

> 문서의 빠른 검색을 위해 설계된 자료구조. 각 샤드는 다수의 세그먼트로 구성되어있음

- Document 저장 시 Elasticsearch는 메모리에 이것들을 모아두고 새로운 세그먼트를 디스크에 기록하여 검색을 리프레쉬(refresh)한다. 
  - 샤드에서 검색 시, 먼저 각 세그먼트를 검색하여 결과를 조합한 후 최종 결과를 해당 샤드의 결과로 리턴
  - 세그먼트는 불변성. 삭제 시 새로운 것을 포인터가 가리킨다. 그리고 특정 임계치를 넘기면 더이상 필요없어진 데이터들을 정리하고 새로운 세그먼트로 병합 한 후 세그먼트를 삭제하며, 이 때 비로소 디스크에서 완전히 삭제

___

## 💿 ****



