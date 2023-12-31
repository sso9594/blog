---
title: "JWT_Login"
date: 2023-08-10T22:19:23+09:00
tags: ["Tech", "JWT","spring boot"]
---

# JWT 토큰 기반 로그인 인증 시스템

<img src="/jwt.png" width=100%>

## JWT란 무엇인가?

JWT는 Json Web Token의 약자입니다. JWT는 당사자 간의 정보를 JSON으로 안전하게 전송하기 위한 Claim기반의 Web Token 입니다. 이 정보는 HMAC 또는 RSA를 사용하는 공개키/개인키를 통해 서명할 수 있기 때문에 신뢰할 수 있습니다. 그래서 해당 토큰을 통해 사용자를 식별하고 리소스에 접근해서 리소스를 가져올 수 있습니다. JWT의 구조는 Header, Payload, Signature 3개로 나뉩니다. 이 3개는 “.” 구분자를 통해 구분됩니다. 그래서 각 부분은 Base64로 인코딩 됩니다. 각 구조에 대해 한번 알아보도록 하겠습니다.

**Header**는 토큰의 타입과 사용중인 서명 알고리즘에 대한 값이 들어가 있습니다. JWT에서 가장 첫번째 부분을 담당 합니다.

**Payload**는 Claim들을 포함하고 있고 JWT에서 두번째 부분을 담당 하고 있습니다. Claim은 사용자에 대한 데이터를 이야기합니다. 그래서 발급자, 토큰 만료시간, 식별자 등을 포함할 수 있습니다.

**Singnature**는 Base64로 인코딩된 Header와 Payload 그리고 secret을 헤더에 있는 알고리즘을 통한 암호화로 생성할 수 있고 중간에 메세지가 변경되지 않았는지 확인하는데 사용됩니다.

Base64를 통해 데이터가 인코딩 되었기 때문에 토큰에는 제3자가 알면 안되는 중요한 정보를 담지 말아야 합니다. 또한 일반적으로 헤더를 통해 전송이 되기 때문에 너무 큰 데이터를 담으면 안됩니다.

![jwt-structure](/jwt-structure.png)

## 회원가입 로직

회원가입 : 아이디 + 비밀번호 + 이메일 + 닉네임 ?

→ 비밀번호는 해쉬함수를 통해 암호화

## 로그인 로직

로그인 : 아이디 + 비밀번호를 입력하면

→ 비밀번호를 해쉬함수를 통해 db의 데이터와 비교

→ 인증이 되면 access token + refresh token을 클라이언트에 전송

→ access token이 끝나기 전에 refresh token을 통해 재발급 요청

기본적인 로그인 로직은 이해했지만, 토큰을 발급하여 **어떻게 서버에서 다시 확인**하고 **재발급은 API화 하여**

**재발급** 하는 건지 **자동으로 클라이언트에서 발급 요청하는 지** 등 좀더 알아봐야 할것 같다. 

## DB 사용

1. 아이디와 암호화된 비밀번호만 redis에 넣고 다른 유저 정보들은 mysql에 넣은 후 매핑
2. mysql에 다 넣기
    
    - 고민중

![loginERD](/loginERD.jpg)
    

- 1번일 경우 이렇게 데이터베이스 구성
- 2번일 경우 login table 과 user table 병합

## JWT 갱신 프로세스

1. User Login시에 accessToken이랑 refreshToken 생성
2. user 테이블에 refreshToken을 저장
3. 시간이 지난뒤에 accessToken 만료
4. token refresh 요청
5. refreshToken으로 user를 조회
6. 조회된 User가 있으면 accessToken이랑 refreshToken 갱신
7. user 테이블에 refreshToken을 저장
8. accessToken과 refreshToken을 response로 반환

## Issue

- 프론트에서 token을 보관할 수 있어야 함.
- token이 발급되어 인가를 받고 재발급이 되는지 어떻게 확인하지
    - 일단 postman으로 해보고 안되면 다시 찾아보기
    - spring test 로 할 수 있나

## 진행상황

- [x]  SHA-256 해쉬 방식을 통한 비밀번호 암호화
- [x]  아이디와 암호화된 비밀번호를 통한 회원가입 API 구현
- [ ]  기본적인 로그인 구현
- [ ]  JWT 기반 Access Token, Refresh Token Provider 구현
- [ ]  로그인 시 Token을 반환하여 인가 시스템 구현
- [ ]  시간 남으면 Email 인증시스템을 통한 회원가입 신뢰도 높히기
