---
title: SIY Spring Cloud Config
author: Jiny
date: 2021-06-08 13:30:00 +0800
categories: [Spring, Cloud]
tags: [spring, cloud, siy, config]
toc: false
---
 
# SIY Spring Cloud Config(RabbitMQ)

___

![image](https://madplay.github.io/img/post/2020-01-30-introduction-to-spring-cloud-config-1.png)

actuator를 통해 원격에서 REST API를 통해 서비스를 Refresh 시킬 수 있습니다.

즉 설정파일이 바뀌었다고 프로젝트를 다시 빌드할 필요가 없어졌습니다.

또한, 실행중에 빌드, 배포 없이 동적으로 변경시킬 수 있습니다.

___

## 💿 **준비**

- RabbitMQ 설치
  - 먼저 Erlang 설치해야됨

___

## 💿 **bootstrap.yml**

bootstrap.yml은 application.yml 보다 먼저 적용되는 yml로 cloud에서 configuration을 가져올 때 사용합니다.

```yml
spring:
  cloud:
    config:
      uri: http://localhost:8888
      name: siyGateway
  profiles:
    active: dev
```

이것은  siyGateway-dev라는 yml파일을 통해 설정하겠다는 것을 의미

___

## 💿 **Exhange Type**

아무것도 설정하지 않았는데 아마도 FanOut방식 인 것 같다.

___

## 💿 **주의 사항**

- @EnableConfigServer를 설정해주어야 한다.
- @RefreshScope: config 저장소에서 설정 파일을 변경했을 때 변경사항을 갱신할 수 있도록 설정하는 것이다.
- actuator는 따로 dependency에 추가해 주어야 한다.
