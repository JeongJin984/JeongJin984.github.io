---
title: Kubernetes 1
author: Jiny
date: 2021-06-27 14:30:00 +0800
categories: [Infra, Kubernetes]
tags: [k8s, kubernetes, infra]
toc: false
---
 
# Kubernetes 1

___

## 💿 **컨트롤 플레인(마스터)**

![image](https://startkubernetes.com/static/ed77ff8be09a6293d24b21f4dadc55d7/5a190/k8s-architecture.png)

![image](https://startkubernetes.com/static/7b7a66152f804ca26d48493975f99124/5a190/k8s-master.png)
_https://startkubernetes.com/blog/k8s_master_and_worker_nodes_


### **kube-api-server**

- 쿠버네티스 시스템 컴포넌트는 오직 API 서버와 통신
- 컴포넌트끼리 서로 직접 통신X
- 때문에 etcd와 통신하는 유일한 컴포넌트 API 서버
- Restful API를 통해 클러스터 상태를 쿼리, 수정할 수 있는 기능을 제공

**구체적 역할**

- 인증 플러그인을 사용한 클라이언트 인증
- 권한 승인 플러그인을 통한 클라이언트 인증
- 승인 제어 플러그인을 통해 요청 받은 리소스를 확인/수정
- 리소스 검증 및 영구 저장

### **큐브 컨트롤러 매니저**

- API 서버는 궁극적으로 아무 역할을 하지 않음
- 컨트롤러에는 다양한 컨트롤러가 존재
- 이 컨트롤러는 API에 의해 받아진 요청을 처리하는 역할
  - replication manager
  - replica set, daemon set, job controller
  - deployment controller
  - stateful set controller
  - node controller
  - service controller
  - end-point contoller
  - namespace controller
  - 영구 볼륨 controller
  - etc...

### **큐브 스케쥴러**

- 일반적으로 실행할 노드를 직접 정해주지 않음
- 요청 받은 리소스를 어느 노드에 실행할지 결정하는 역할
- 현재 노드의 상태를 점검하고 최상의 노드를 찾아 배치
- 다수의 포드를 배치하는 경우에는 라운드로빈을 사용하여 분산

### **etcd**

> A distributed, reliable key-value store for the most critical data of a distributed system

- 클러스터의 데이터베이스 역할을 하는 서버
- 클러스터의 모든 설정, '상태' 데이터를 저장하는 부분(key-value 형태로 저장)
  - 즉 etcd만 잘 백업하면 언제든지 클러스터 복구 가능
- 여러개로 분산해서 복제가능, 안전성 높음, 속도 빠름
- rw 뿐만 아니라 watch 기능으로 상태변경 체크 가능
- 오직 API 서버와 통신하고, 다른 모듈은 API 서버를 거쳐서 etcd 데이터에 접근함

___

## 💿 **POD**

![image](https://www.slipp.net/wiki/download/attachments/41583706/image2019-9-16_16-5-44.png?version=1&modificationDate=1568624763000&api=v2)

- k8s의 공동 배포된 그룹이며 쿠버네티스의 기본 빌딩 블록을 대표
- k8s는 컨테이너를 개별적으로 배포하지 않고 컨테이너의 포드를 항상 배포하고 운영
- 일반적으로 포드는 단일 컨테이너만 포함하지만 다수의 컨테이너를 포함 할 수 있음
- 포드는 다수의 노드에 생성되지 않고 단일 노드에서만 실행
- 여러 프로세스를 실행하기 위해서는 컨테이너 당 단일 프로세스가 적합
- 다수의 프로세스를 제어하려면 -> 다수의 컨테이너를 다룰 수 있는 그룹이 필요

### **장점**

- 밀접하게 연관된 프로세스를 함께 실행하고 마치 하나의 환경에서 동작하는 것처럼 보임
- 동일한 환경을 제공하면서 다소 격리된 상태로 유지
 
### **동일한 포드의 컨테이너 사이의 부분 격리**

- 포드의 모든 컨테이너는 동일한 네트워크 및 UTS 네임스페이스에서 실행
- 같은 호스트 이름 및 네트워크 인터페이스를 공유(포드 충돌 가능성 있음)
- 포드의 모든 컨테이너는 동일한 IPC 네임스페이스 아래에서 실행되며 IPC를 통해 통신 가능
  - 최근에는 동일한 PID 네임스페이스를 공유할 수 있지만 이 기능은 기본적으로 활성화 되있지 않음

### **YML에서의 포드 정의**

> apiVersion, kind, meta-data, spec, satus로 구성

- 구성요소
  - apiVersion: k8s api 버전을 가리킴
  - kind: 어떤 리소스 유형인지 결정(Pod-Replica-Controller, Service, Pod 등등)
  - 메타데이터: 포드와 관련된 이름, 네임 스페이스, 라벨, 그 밖의 정보 존재
  - 스펙: 컨테이너, 볼륨 등의 정보
  - 상태: 포드의 상태, 각 컨테이너의 설명 및 상태, 포드 내부의 IP 및 그 밖의 기본 정보 등

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```

___

## 💿 **레이블과 셀렉터**

### **레이블**

![image](https://raw.githubusercontent.com/act-coe/act-coe.github.io/master/images/k8s/chapter3/figure3.6.png)

> 포드를 그룹별로 구별하기 위한 값

- 모든 리소스를 시작하는 매우 간단하면서도 강력한 쿠버네티스 기능
- 리소스에 첨부하는 임의의 key-value 쌍
- 레이블에 셀렉터를 사용하면 각종 리소스를 필터링하여 선택할 수 있음
- 리소스는 한 개 이상의 레이블을 가질 수 있음
- 리소스를 만드는 시점에 레이블을 첨부
- 기존 리소스에도 레이블의 값을 수정 및 추가 기능
- 모든 사람이 쉽게 이해 할 수 있는 체계적인 시스템 구축 가능
  - app: app 구성요소, 마이크로 서비스 유형 지정
  - rel: 어플리케이션 버전 지정

```yml
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

- 새로운 레이블 추가: label 명령어
  - kubectl label pod http-go-v2 test=foo
- 기존 레이블을 수정: --overwrite 옵션을 줘서 실행
  - kubectl label pod http-go-v2 rel-beta --overwrite
- 레이블 보기
  - kubectl get pod --show-labels
- 특정 레이블 칼럼
  - kubectl get pod -L app, rel

___

## 💿 **Namespace**

- 리소스를 각각의 분리된 영역으로 나누기 좋은 방법
- 여러 네임스페이스를 사용하면 복잡한 쿠버네티스 시스템을 더 작은 그룹으로 분할
- Multi Tenant 환경을 분리하여 리소스를 생산, 개발, QA 환경으로 사용
- 리소스 이름은 네임스페이스 내에서만 고유 명칭 사용

### **네임스페이스 상세 확인**

- kubectl get을 옵션없이 사용하면 default 네임스페이스에 질의
- 다른 사용자와 분리된 환경으로 타인의 접근 제한
- 네임스페이스 별로 리소스 접근 허용과 리소스 양도 제어 가능
- --namespace 나 -n 을 사용하여 네임스페이스 별로 확인 가능