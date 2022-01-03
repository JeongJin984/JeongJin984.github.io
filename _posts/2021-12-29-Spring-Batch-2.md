---
title: Spring Batch 2
author: Jiny
date: 2021-12-06 13:09:00 +0800
categories: [Java, Spring]
tags: [java, spring, aop]
toc: false
---

# Spring Batch 2
___

## **💿 배치 초기화 설정**

### **JobLauncherApplication**

- Spring Batch 작업을 시작하는 ApplicationRunner로서 BatchAutoConfiguraion에서 생성됨
- Spring Boot에서 제공하는 ApplicationRunner의 구현체로 App이 정상적으로 구동되자 마자 실행됨
- 기본적으로 빈으로 등록된 모든 job을 실행시킨다.

### **BatchProperties**

- Spring Batch의 환경 설정 클래스
- Job 이름, 스키마 초기화 설정, 테이블 Prefix 등의 값을 설정할 수 있다.
- application.properties or application.yml 파일에 설정함


```yml
batch:
  job:
    names: ${job.name: NONE}
  initialize-schema: Never
  tablePrefix: SYSTEM
```

### **Job 실행 옵션**

- 지정한 Batch Job만 실행하도록 할 수 있음
- spring.batch.job.names: ${job.name: NONE}
- App 실행시 Program arguments로 job 이름 입력한다.
  - --job.name=hello.job
  - --job.name=helloJob, simpleJob (하나 이상의 job을 실행 할 경우 쉼표로 구분해서 입력함)
