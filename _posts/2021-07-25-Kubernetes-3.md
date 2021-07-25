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