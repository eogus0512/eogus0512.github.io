---
layout: post
title: (OAuth2 Login) 구글 소셜 로그인 구현 (3) - JWT 인증
subtitle: Spring Security + JWT를 이용한 소셜 로그인 구현
categories: OAuth2
tags: [Spring, Spring Security, JWT]
---

## JWT
### JWT란?
- JSON Web Token의 약자로 인증에 필요한 정보들을 암호화한 JSON 토큰을 의미한다.
- 토큰에는 사용자 관련 여러 정보가 포함되어있다.
- JWT를 통해 Stateless 하게 설계가 가능하다.

### JWT 인증 순서
1. 사용자가 로그인 정보를 가지고 로그인을 서버에 요청한다.
2. 서버는 JWT 토큰을 생성하여 클라이언트에 보낸다.
3. 클라이언트는 받은 JWT 토큰을 사용하여 서비스 요청할 때마다 Http Header에 JWT를 넣어 서버에 요청한다.
4. 서버는 해당 JWT를 검증하고, 유효하다면 요청에 대한 응답을 반환한다.

### JWT 구조
- Header
  - 토큰의 타입과 전자서명 알고리즘이 저장된다.
- Payload
  - Claim운 토큰에서 사용하는 정보를 포함한다.
  - Payload는 이러한 Claim을 여러개 포함한다.
- Signature
  - header와 payload를 암호화한 것과 서버가 가지고 있는 개인 키를 암호화한 정보가 저장된다.
 
### Access Token & Refresh Token
- Access Token
  - 로그인 시 발급 받고, 인증 처리 위해 사용되는 토큰을 말한다.
  - 탈취 위험으로부터 벗어나기 위해 유효 기간을 짧게 설정한다.
  - 처음 로그인 요청 시 서버에서 Access Token을 발급하고, 클라이언트는 Http Header에 이를 저장하여 서비스를 이용하기 위해 요청마다 Access Token을 보낸다.
- Refresh Token
  - 유효시간이 짧은 Access Token으로 인한 trade-off 문제를 해결하기 위한 토큰
  - DB와 같은 저장소에서 저장되어 Access Token을 재발급 해주는 토큰을 말한다.
  - 처음 로그인 요청시 서버에서 Access Token과 함께 Refresh Token을 발급하는데 Refresh Token은 Http Header에 저장되지 않고, 서버 DB에 저장된다.
  - Refresh Token은 자체적으로 인증 용도로 사용되지 않고, Access Token을 재발급하기 위한 용도로만 사용된다.
 
<br>
<br>
