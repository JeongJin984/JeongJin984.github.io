---
title: Spring Batch 1
author: Jiny
date: 2021-12-06 13:09:00 +0800
categories: [Java, Spring]
tags: [java, spring, aop]
toc: false
---

# Spring Batch 1
___

## **💿 DB Schema**

### **Job 관련 테이블**

- **BATCH_JOB_INSTANCE**
  - Job이 실행될 때 JobInstance 정보가 저장되며 job_name과 Job_key을 키로 하여 하나의 데이터가 저장
  - 동일한 job_name 과 job_key로 중복 저장될 수 없다.
  - 
- **BATCH_JOB_EXECUTION**
  - job의 실행정보가 저장되며 Job 생성, 시작, 종료시간, 실행상태, 메시지 등을 관리
- **BATCH_JOB_EXECUTION_PARAMS**
  - Job과 함께 실행되는 JobParameter 정보를 저장
- **BATCH_JOB_EXECUTION_CONTEXT**
  - Job의 실행동안 여러가지 상태정보, 공유 데이터를 직렬화(Json 형식) 해서 저장
  - Step 간 서로 공유 가능함

### **Step 관련 테이블**

- **BATCH_STEP_EXECUTION**
  - Step의 실행정보가 저장되며 생성, 시작 종료시간, 실행 상태, 메시지 등을 관리
- **BATCH_STEP_EXECTION_CONTEXT**
  - Step의 실행동안 여러가지 상태정보, 공유 데이터를 직렬화(Json)해서 저장
  - Step 별로 저장되며 Step간 서로 공유할 수 없음
- 

___

## **💿 Job**

![image](https://docs.spring.io/spring-batch/docs/1.1.x/spring-batch-docs/reference/html/images/spring-batch-reference-model.png)

> 배치 계층 구조에서 가장 상위에 있는 개념으로서 하나의 배치 작업 자체를 의미함

- API 서버의 접속 로그 데이터를 통계 서버로 옮기는 배치 -> Job 그 자체
- Job Configuration을 통해 생성되는 객체 단위로서 배치작업을 어떻게 구성하고 실행할 것인지 전체적으로 설정하고 명세해 놓은 객체
- 배치 Job을 구성하기 위한 최상위 인터페이스이며 스프링 배치가 기본 구현체를 제공
- 여러 Step을 포함하고 있는 컨테이너로서 반드시 한개 이상의 Step으로 구성해야 함

### 기본 구현체

- **Simple Job**
  - 순차적으로 Step을 실행시키는 Job
  - 모든 Job에서 유용하게 사용할 수 있는 표준 기능을 갖고 있음
- **Flow Job**
  - 특정한 조건과 흐름에 따라 Step을 구성하여 실행시키는 Job
  - Flow 객체를 실행시켜서 작업을 진행함
- 

____

## **💿 JobInstance**

> Job이 실행될 때 생성되는 Job의 논리적 실행 단위 객체로서 고유하게 식별 가능한 작업 실행을 나타냄

- Job의 설정과 구성은 동일하지만 Job이 실행되는 시점에 처리하는 내용은 다르기 때문에 Job의 실행을 구분해야 함
  - 예를 들어 하루에 한번 씩 배치 Job이 실행된다면 매일 실행되는 각각의 Job을 JobInstance로 표현합니다.
- JobInstance 생성 및 실행
  -  처음 시작하는 Job + JobParameter(해쉬)일 경우 새로운 JobInstance 생성
  -  이전과 동일한 Job + JobParameter일 경우 이미 존재하는 JobInstance 리턴
  - Job과는 1:M 관계
  
### **BATCH_JOB_INSTANCE 테이블과 매핑**

- JOB_NAME(Job)rhk JOB_KEY(JobParameter 해시값)가 동일한 데이터는 중복해서 저장할 수 없음

___

## **💿 JobParameter**

> Job을 실행할 때 함께 포함되어 사용되는 파라미터를 가진 도메인 객체

- 하나의 Job에 존재할 수 있는 여러개의 JobInstance를 구분하기 위한 용도
- JobParameters와 Job Instance는 1:1 관계

__

## **💿 JobExecution**

> JobInstance에 대한 한번의 시도를 의미하는 객체로서 Job 실행 중에 발생한 정보들을 저장하고 있는 객체

- 시작시간, 종료시간, 상태(시작됨, 완료, 실패)종료 상태의 속성을 가짐
- JobExecution은 FAILED 또는 COMPLETED 등의 Job의 실행 결과 상태를 가지고 있음
- JobExecution의 실행 상태 결과가 COMPLETED면 JobInstance 실행이 완료된 것으로 간주해서 재 실행이 불가함
- JobExecution의 실행 상태 결과가 FAILED면 JobInstance 실행이 완료되지 않은 것으로 간주해서 재실행이 가능함
  - JobParameter가 동일한 값으로 Job을 실행할지라도 JobInstance를 계속 실행할 수 있음
- JobExecution의 실행 상태 결과가 COMPLETED가 될 때까지 하나의 JobInstance 내에서 여러번의 시도가 생길 수 있음
- JobInstance와 JobExecution는 1:M의 관계로서 JobInstance에 대한 성공/실패의 내역을 가지고 있음

___

## **💿 Step**

> Batch Job을 구성하는 독립적인 하나의 단계로서 실제 배치 처리를 정의하고 컨트롤하는 데 필요한 모든 정보를 가지고 있는 도메인 객체

- 단순한 단일 태스크 뿐 아니라 입력과 처리 그리고 출력과 관련된 복잡한 비즈니스 로직을 포함하는 모든 설정들을 담고 있다.
- 배치작업을 어떻게 구성하고 실행할 것인지 Job의 세부 작업을 Task 기반으로 설정하고 명세해 놓은 객체
- 모든 Job은 하나 이상의 step으로 구성됨

### **기본 구현체**

- TaskletStep
  - 가장 기본이 되는 클래스로서 Tasklet 타입의 구현체들을 제어한다.
- PartitionStep
  - 멀티 스레드 방식으로 Step을 여러 개로 분리해서 실행한다.
- JobStep
  - Step 내에서 Job을 실행하도록 한다.
- FlowStep
  - Step  내에서 Flow를 실행하도록 한다.

___

## **💿 StepExecution**

> Step에 대한 한번의 시도를 의미하는 객체로서 Step 실행 중에 발생한 정보들을 저장하고 있는 객체

- 시작시간, 종료시간, 상태, commit count, rollback count 등의 속성을 가짐
- Step이 매번 시도될 때마다 생성되며 각 Step 별로 생성된다.
- Job이 재시작 하더라도 이미 성공적으로 완료된 Step은 재 실행되지 않고 실패한 Step만 실행된다.
- 이전 단계 Step이 실패해서 현재 Step을 실행하지 않았다면 StepExecution을 생성하지 않는다. Step이 실제로 시작됐을 때만 StepExecition을 생성한다.
  
### **Job Execution 과의 관계**

- Step의 StepExecution이 모두 정상적으로 완료 되어야 JobExecution이 정상적으로 완료된다.
- Step의 StepExecution 중 하나라도 실패하면 JobExecution은 실패한다.

### **BATCH_STEP_EXECUTION 테이블과 매핑**

- JobExecution과 StepExecution은 1:M 관계
- 하나의 Job에 여러 개의 Step으로 구성했을 경우 각 StepExecution은 하나의 jobExecution을 부모로 가진다.

___

## **💿 StepContribution**

> Chunk 프로세스의 변경 사항을 버퍼링 한 후 StepExecution 상태를 업데이트하는 도메인 객체

- 청크 커밋 직전에 StepExecution의 apply 메서드를 호출하여 상태를 업데이트 함
- ExitStatus의 기본 종료코드 외 사용자 정의 종료코드를 생성해서 적용할 수 있음

___

## **💿 StepContribution**

> 프레임워크에서 유지 및 관리하는 키/값으로 된 컬렉션으로 StepExecution 또는 JobExecution 객체의 상태를 저장하는 공유 객체

- DB에 직렬화 한 값으로 저장됨 - ("key" : "value")
- 공유 범위
  - Step 범위 - 각 Step의 StepExecution에 저장되며 Step간 서로 공유 안됨
  - Job 범위 - 각 Job의 JobExecution에 저장되며 Job간 서로 공유 안되며 해당 Job의 Step 간 서로 공유됨
- Job 재 시작시 이미 처리한 Row 데이터는 건너뛰고 이후로 수행하도록 할 때 상태 정보를 활용한다.

___

## **💿 JobRepository**

> 배치 작업 중의 정보를 저장하는 저장소 역할

- Job이 언제 수행되었고 언제 끝났으며, 몇번 실행되었고 실행에 대한 결과 등의 배치 작업의 수행과 관련된 모든 meta data를 저장
  - JobLauncher, Job, Step 구현체 내부에서 CRUD 기능을 처리함(DB와 연결된 부분이라는 뜻)

### **JobRepository 설정**

- @EnableBatchProcessing 어노테이션만 선언하면 JobRepository가 자동으로 빈으로 생성됨
- BatchConfigurer 인터페이스를 구현하거나 BasicBatchConfigurer를 상속해서 JobRepository 설정을 커스터마이징 할 수 있다.
  - **JDBC 방식으로 설정 - JobRepositoryFactoryBean**
    - 내부적으로 AOP 기술을 통해 트랜잭션 처리를 해주고 있음
    - 트랜잭션 isolation의 기본값은 SERIALIZABLE로 최고 수준, 다른 레벨(READ_COMMITED, REPEATABLE_READ)로 지정 가능
    - 메타테이블의 Table Prefix를 변경할 수 있음, 기본 값은 "BATCH_"임
  - **In Memory 방식으로 설정 - MapJobRepositoryFactoryBean**
    - 성능 등의 이유로 도메인 오브젝트를 굳이 데이터베이스에 저장하고 싶지 않을 경우
    - 보통 Test나 프로토타입의 빠른 개발이 필요할 때 사용  

___

## **💿 JobLauncher**

> 배치 Job을 실행시키는 역할을 한다.

- Job과 Job Parameters를 인자로 받으며 요청된 배치 작업을 수행한 후 최종 Client에게 JobExecution을 반환
- Spring Boot Batch가 구동이 되면 JobLauncher 빈이 자동 생성 된다.
- Job 실행
  - JobLauncher.run(Job, JobParameter)
  - Spring Boot Batch는 JobLauncherApplicationRunner가 자동적으로 JobLauncher를 실행시킨다.
  - 동기적 실행
    - taskExecutor를 SyncExecutor로 설정할 경우(기본)
    - JobExecution을 획득하고 배치 처리를 최종 완료한 후 Client에게 JobExecution을 반환
    - 스케줄러에 의한 배치처리에 적합 함 - 배치 처리 시간이 길어도 상관없는 경우
  - 비동기적 실행
    - taskExecutor가 SimpleAsyncTaskExecutor로 설정할 경우
    - JobExecution을 획득한 후 Client에게 바로 jobExecution을 반환하고 배치처리를 완료한다.
    - HTTP 요청에 의한 배치처리에 적합함 - 배치처리 시간이 길 경우 응답이 늦어지지 않도록 함



