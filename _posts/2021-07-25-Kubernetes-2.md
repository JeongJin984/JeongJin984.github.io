---
title: Kubernetes 2
author: Jiny
date: 2021-07-25 14:30:00 +0800
categories: [Infra, Kubernetes]
tags: [k8s, kubernetes, infra]
toc: false
---
 
# Kubernetes 2

___

## 💿 **Liveness, Readiness, Startup Probes**

### **Liveness Probes**

- 컨테이너가 살아있는지 판단하고 다시 시작하는 기능
- 컨테이너의 상태를 스스로 판단하여 교착 상태에 빠진 컨테이너를 재시작
- 버그가 생겨도 높은 가용성을 보임

### **Readiness Probes**

- 포드가 준비된 상태에 있는지 확인하고 정상 서비스를 시작하는 기능
- 포드가 적절하게 준비되지 않은 경우 로드밸런싱을 하지 않음

### **Startup Probes**

- 애플리케이션 시작 시기를 확인하여 가용성을 높이는 기능
- Liveness와 Readiness의 기능을 비활성화
- 시작 시 과도하게 위의 두 Probe 가 동작하여 Pod이 생성되지 않을 수 있으므로 시작 시간을 벌어주는 용도
  - 부팅이 오래걸리는 컨테이너가 포함될 수도 있기 때문

___

## 💿 **Liveness Probe**

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```
- Liveness Probe는 커맨드 실행(Http 요청이나 리눅스 명령)으로 컨테이너 확인
  - 위의 yml은 Http 요청으로 확인
    - 200이상 400미만(컨테이너 유지)
    - 서버 응답 코드가 그 외일 경우(컨테이너 재시작)

___

## 💿 **Readiness Probe**

```yml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

- Readiness TCP 설정
  - 준비 프로브는 8080포트 검사
  - 5초 후 검사 시작
  - 검사 주기 10초
    - 서비스를 시작해도 된다
- Liveness TCP 설정
  - 활성화 프로브는 8080 포트 검사
  - 14초 후 검사 시작
  - 검사 주기 20초
    - 컨테이너를 유지한다.

___

## 💿 **Startup Probe**

```yml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```

- 시작할 때까지 검사를 수행
- http 요청을 통해 검사
- 30번을 검사하여 10초 간격으로 수행
- 300초 후에도 포드가 정상적으로 동작하지 않는 경우 종료
  - 즉 300초 동안 포드가 정상 실행될 시간을 벌어준다.

___

## 💿 **레플리케이션 셋(레플리케이션 컨트롤러)**

> - 포드가 항상 실행되도록 유지하는 쿠버네티스 리소스
> - 노드가 클러스터에서 사라지는 경우 해당 포트를 감지하고 대체 포드 생성
> - 실행 중인 포드의 목록을 지속적으로 모니터링 하고 '유형'의 실제 포드 수가 원하는 수와 일치하는지 확인

- 세가지 요소
  - R.C가 관리하는 포드의 범위를 결정하는 레이블 셀렉터
  - 실행해야 하는 포드의 수를 결정하는 복제본의 수
  - 새로운 포드의 모양을 설정하는 포드 템플릿
- 장점
  - 포드가 없을 경우 새 포드를 항상 실행
  - 노드가 장애 발생 시 다른 노드에 복제본 생성
  - 수동, 자동으로 수평 스케일링

```yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

___

