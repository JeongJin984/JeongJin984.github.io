---
title: SIY Spring Cloud Api GateWay 2
author: Jiny
date: 2021-06-08 12:30:00 +0800
categories: [Spring, Cloud]
tags: [spring, cloud, siy, apigateway]
toc: false
---
 
# Spring Cloud Api Gateway(인증 및 인가)

___

## 💿 **CORS**

API Gateway도 cors policy가 적용 되므로 defaultfilter를 적용해 줌으로서 허용하는 origin을 설정할 수 있다.

___

## 💿 **인증 방식**

- 쿠키를 이용한 jwt 인증
- redis의 Cache를 이용한 logout

___

## 💿 **인증 하는 곳**

> gateway 코드: https://github.com/JeongJin984/SIYApiGateway

> front 코드: https://github.com/JeongJin984/KitMarketFront

- Front End
  - 자동로그인을 위해 접속시 "/home" 과 "/login", "/signup"을 제외한 모든 곳에서 user 정보를 load(SSR)한다.
  - 위의 url에서 access 토큰 만료 시 refresh를 체크해서 만료되지 않았으면 access 토큰을 refresh토큰을 이용해서 새로 발급받는 요청을 보낸다.(loadJWT 라는 미들웨어를 이용했다.)
- Back End
  - login과 signup을 제외한 모든 요청은 Filter를 통해서 JWT를 인증 받아야만 resource를 얻을 수 있다.

___

## 💿 **로그인**

로그인은 Spring Secuirty를 사용하여 일반적으로 로그인

자세한 사항은 Security 편 참고

___

## 💿 **로그 아웃**

### **Redis 설정**

1. Redis 설치
2. Redis 연결(application.yml)
3. Redis Config 클래스로 설정
   - RedisTemplate 설정
     - KeySerializer
     - ValueSerializer 
   - ConnectionFactory 설정
     - Host
     - Port
4. Cache Config 클래스로 설정 

```java
@Configuration
@EnableRedisRepositories
public class CacheConfig {

    @Bean
    RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
        return (builder) -> {
            Map<String, RedisCacheConfiguration> configurationMap = new HashMap<>();
            configurationMap.put("tokenCache",
                    RedisCacheConfiguration.defaultCacheConfig()
                            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
            );
            builder.withInitialCacheConfigurations(configurationMap);
        };
    }
}
```
이 부분이 Cache 설정인데 configuration.put()에서 "tokenCahe 부분이 캐시의 키가 되며 이 키를 가지고 캐시를 집어 넣는다.

5. Repository 만들기(JPA처럼 interface로 미리 만들어진 함수를 사용할 수 있다.)
6. Service 만들기, 캐시 시간 설정 하기

```java
@Service
@RequiredArgsConstructor
public class ExpiredTokenService {

    private final ExpiredTokenRepository repository;

    @Cacheable(value = "tokenCache", key = "#token")
    public ExpiredToken addToken(String token) {
        return repository.save(new ExpiredToken(token));
    }

    @Cacheable(value = "tokenCache", key = "#token" ,condition="#id!=null")
    public ExpiredToken isValid(String token) {
        return repository.findFirstByValue(token);
    }
}
```

이런 식으로

캐시 저장 시간은 entity의 timetolive로 설정했다.(초단위로 재는 듯 하다. 즉 20초로 설정)
```java
@RedisHash("expiredToken")
@NoArgsConstructor
public class ExpiredToken {
    @Id
    private Long id;

    @Indexed
    private String value;

    @TimeToLive
    private Long expiration = 20L;

    public ExpiredToken(String value) {
        this.value = value;
    }
}
```