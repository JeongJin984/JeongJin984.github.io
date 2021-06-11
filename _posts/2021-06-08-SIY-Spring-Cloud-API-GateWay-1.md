---
title: SIY Spring Cloud Api GateWay 1
author: Jiny  
date: 2021-06-08 11:30:00 +0800
categories: [Spring, Cloud]
tags: [spring, cloud, apigateway, siy]
toc: false
---
 
# Spring Cloud Api Gateway(Filter)
___

## 💿 **APi Gateway 주요 기능**

1. 인증 및 인가(Authentication and Authorization)

> 인증서 관리나, 인증, SSL, 프로토콜 변환과 같은 기능들은 API Gateway에서 오프로드 함으로, 각각의 서비스의 부담을 줄이고, 서비스의 관리 및 업그레이드를 보다 쉽게 처리

> **Authentication(인증)과 Authorization(인가)의 차이**
>
> Authentication은 유저가 누구인지 확인하는 절차(A라고 하며 접근하는 사람이 진짜 A인지 확인하는 절차)이고, Authorization은 어떠한 유저가 특정 자원에 접근하려 할때, 그에대한 접근 권한이 있는지 확인하는 절차입니다.

2. 요청 절차의 단순화

>  API Gateway는 여러 클라이언트의 요청을 단일 클라이언트의 요청으로 대체 가능하도록 합니다. 따라서 클라이언트와 백엔드 간의 API 통신량을 줄여주어 대기시간을 줄이고 효율성을 높여줄 수 있습니다.

3. 라우팅 및 로드밸런싱

> API Gateway는 클라이언트로 부터 접수된 메세지에 따라, API 호출을 적절한 서비스에 라우팅 할 수 있는 기능이 있습니다. 또한, 서비스 인스턴스들에 대한 부하분산을 할 수 있습니다.

4. 서비스 오케스트레이션

> 오케스크레이션은 여러개의 마이크로 서비스를 묶어 새로운 서비스를 만드는 개념입니다. 오케스트레이션 로직을 과도하게 넣는 것은, API Gateway의 부담을 늘리는 것으로, 성능 저하를 일으킬 수 있어, MSA와 API Gateway에 대한 높은 수준의 기술적 이해를 바탕으로 이루어져야 합니다.

5. 서비스 디스커버리

> 클라우드 환경에서는 동적인 환경에서 배포되기 때문에 서비스의 위치를 찾는 것이 어렵습니다. 이러한 서비스의 위치(IP 주소와 포트번호)를 찾는 것을 "Service Discovery"라 하며, API Gateway에서는 서버사이드나, 클라이언트 사이드를 기준으로 하여 서비스 디스커버리를 구현할 수 있습니다.

___

## 💿 **APi Gateway 고려 사항**

1. Api Gateway를 내부 ms와 결합한다는 것: SOA에서의 ESB문제에서 발생했던 문제점이 다시 발생 가능
   - ESB 내부 처리 로직을 XML을 기반으로 하였는데, XML의 파싱은 오버헤드가 큰 작업
   - 과도한 Orchestration 등 무거운 로직을 가지고 있었습니다. 특히 ESB를 Gateway로의 특성이 아닌 시스템을 통합하기 위한 역할로 많이 구현하였기 때문에 많은 실패 사례가 발생
2. Gateway의 Scale-Out 적용에 원할하지 않을 경우 Api Gateway가 병목지점이 될 가능성
3. 추가 적인 gateway라는 계층 생성으로 인한 속도 저하

___

## 💿 **Spring Cloud Gateway**

- Spring Cloud Eureka Server(Discovery Service)와 결합
- 인증 및 인가 서비스

### **Filter**

**종류**

- GlobalFilter
- CustomFilter
  - PreFilter
  - PostFilter

**CustomFilter 생성**

```java
@Component
@Slf4j
public class CustomAuthFilter extends AbstractGatewayFilterFactory<CustomAuthFilter.Config> {
    public CustomAuthFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return ((exchange, chain) -> {
           ServerHttpRequest request = exchange.getRequest();
           ServerHttpResponse response = exchange.getResponse();

           if(config.isPreLogger()) {
              log.info("PreLogger");
           }

           return chain.filter(exchange).then(Mono.fromRunnable(() -> {
              if(config.isPostLogger()) {
                 log.info("PostLogger");
              }
           }));
        });
    }

    @Data
    public static class Config {
       private String baseMessage;
       private boolean preLogger;
       private boolean postLogger;
    }
}
```

**CustomFilter 등록**

```yml
server:
  port: 8000

spring:
  application:
    name: gateway-service
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user/**
          filters:
            - RewritePath=/user/?(?<segment>.*), /$\{segment}
            - name: CustomAuthFilter
              args:
                 baseMessage: Hi, there.
                 preLogger: true
                 postLogger: true

        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/order/**
          filters:
            - RewritePath=/user/?(?<segment>.*), /$\{segment}
```

- RewritePath: Gateway로 요청이 들어오면 Path 값을 자동으로 재작성
- predicates: Gateway로 들어온 요청에 따라 Load balancing
  - 예를 들자면 /order/** (오더로 들어온 요청 은 ServiceDiscovery에서 ORDER-SERVICE의 URL 동적으로 찾아서 요청을 보냄)
  - lb란 service Discovery에서 URL을 검색하겠다는 뜻
- name: filter를 preFilter와 postFilter를 구분해서 사용하고 싶다면 name을 붙혀 Filter를 정의하고 args를 초기화하는 부분을 작성해야 함

**GlobalFilter 생성**

```java
@Component
@Slf4j
public class GlobalFilter extends AbstractGatewayFilterFactory<GlobalFilter.Config> {
    public CustomAuthFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return ((exchange, chain) -> {
           ServerHttpRequest request = exchange.getRequest();
           ServerHttpResponse response = exchange.getResponse();

           if(config.isPreLogger()) {
              log.info("PreLogger");
           }

           return chain.filter(exchange).then(Mono.fromRunnable(() -> {
              if(config.isPostLogger()) {
                 log.info("PostLogger");
              }
           }));
        });
    }

    @Data
    public static class Config {
       private String baseMessage;
       private boolean preLogger;
       private boolean postLogger;
    }
}
```

**GlobalFilter 등록**

```yml
spring:
  cloud:
    gateway:
      default-filters:
        - name: GlobalFilter
          args:
            baseMessage: Spring Cloud GlobalFilter
            preLogger: true
            postLogger: true
    routes: ...
```