---
title: authentication-jwt
author: Jiny
date: 2021-06-08 09:30:00 +0800
categories: [Authentication, Cloud]
tags: [authentication, security, jwt, token]
toc: false
---
 
# authentication(JWT)
___

## 💿 **jwt**

### **구조**

- WT는 Header, Payload, Signature의 3 부분으로 이루어지며, Json 형태인 각 부분은 Base64로 인코딩 되어 표현된다.
- 각각의 부분을 이어 주기 위해 . 구분자를 사용
- Base64는 암호화된 문자열이 아니고, 같은 문자열에 대해 항상 같은 인코딩 문자열을 반환(즉 암호화 안되있어서 공개되어있다.)

- **Header(헤더)**
  - typ 와 alg 두가지 정보로 구성(Signature를 해싱하기 위한 알고리즘 지정)
    - typ: 토큰의 타입 지정(JWT)
    - alg: 알고리즘 방식을지정하며 서명(signature) 및 토큰 검증에 사용
- **payload(페이로드)**
  - 토큰에서 사용할 정보의 조각(Claim)이 담겨있음
  - Registered Claim(등록된 클레임): 토큰 정보를 표현하기 위해 이미 정해진 종류의 데이터로 구성
    - iss: 토큰 발급자(issuer)
    - sub: 토큰 제목(subject)
    - aud: 토큰 대상자(audience)
    - exp: 토큰 만료시간(NumericDate 형식)
    - nbf: 토큰 활성 날짜(not before) 이 날의 지나기 전의 토큰은 활성화 안됨
    - iat: 토큰 발급 시간(issedat) 토큰 발급 이후의 경과 시간
    - jti: 토큰 식별자(jwt id) 중복 방지를 위해 사용하며 일회용 토큰 등에 사용
  - Public Claim(공개 클레임): 사용자 정의 클레임으로, 공개용 정보를 위해 사용(충돌 방지를 위해 URI 포맷)
  - Private Claim(비공개 클레임): 사용자 정의 클레임으로, 서버와 클라이언트 사이에 임의로 지정한 정보를 저장
    - token type이나 username 등등
- **Signature(서명)**: 토큰을 인코딩하거나 유효성 검증을 할 때 사용하는 고유한 암호화 코드
  - 헤더(Header)와 페이로드(Payload)의 값을 각각 BASE64로 인코딩
  - 비밀키를 이용해 헤더에 정의한 알고리즘으로 해싱, 이 값을 다시 BASE64로 인코딩

### **고려사항**

- **Self-contained**: 토큰 자체에 정보를 담고 있으므로 양날의 검이 될 수 있다.
- **토큰 길이**: 토큰의 페이로드(Payload)에 3종류의 클레임을 저장하기 때문에, 정보가 많아질수록 토큰의 길이가 늘어나 네트워크에 부하를 줄 수 있다.
- **Payload 인코딩**: 페이로드(Payload) 자체는 암호화 된 것이 아니라, BASE64로 인코딩 된 것이다. 중간에 Payload를 탈취하여 디코딩하면 데이터를 볼 수 있으므로, JWE로 암호화하거나 Payload에 중요 데이터를 넣지 않아야 한다.
- **Stateless**: JWT는 상태를 저장하지 않기 때문에 한번 만들어지면 제어가 불가능하다. 즉, 토큰을 임의로 삭제하는 것이 불가능하므로 토큰 만료 시간을 꼭 넣어주어야 한다.
- **Tore Token**: 토큰은 클라이언트 측에서 관리해야 하기 때문에, 토큰을 저장해야 한다.
