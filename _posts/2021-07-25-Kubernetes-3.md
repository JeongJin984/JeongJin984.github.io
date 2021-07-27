---
title: Kubernetes 3
author: Jiny
date: 2021-07-25 15:30:00 +0800
categories: [Infra, Kubernetes]
tags: [k8s, kubernetes, infra]
toc: false
---
 
# Kubernetes 3

___

## 💿 **Deployment**

> - 어플리케이션을 다운 타임없이 업데이트 가능하도록 도와주는 리소스
> - 레플리카 셋과 레플리케이션 컨트롤러 상위에 배포되는 리소스

- 모든 포드를 업데이트하는 방법
  - 잠깐의 다운 타임 발생(새로운 포드를 생성시키고 작업이 완료되면 오래된 포드를 삭제)
  - 롤링 업데이트

### **롤링 업데이트**

- 기존의 모든 포드를 삭제 후 새로운 포드 생성
  - 잠깐의 다운 타임 발생
- 새로운 포드를 실행시키고 작업이 완료되면 오래된 포드를 삭제
  - 새 버전을 실행시키는 동안 구 버전 포드와 연결
  - 서비스의 레이블 셀렉터를 수정하여 간단하게 수행 가능
- 레플리케이션 컨트롤러가 제공하는 롤링 업데이트
  - 이전에는 kubectl을 사용한 스케일링을 사용하여 수동으로 롤링 업데이트 진행 가능
  - kubectl 중단되면 업데이트는?
  - 레플리케이션 컨트롤러 또는 레플리카 셋을 통제할 수 있는 시스템이 필요
  - 디플로이먼트 파드가 이러한 역할을 수행

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: app-go
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-go
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: http-go:v1
        image: gasbugs/http-go:v1 
        ports:
        - containerPort: 80
```

- kubectl create -f nginx-deployment.yml --record=true
  - log를 남겨놓음