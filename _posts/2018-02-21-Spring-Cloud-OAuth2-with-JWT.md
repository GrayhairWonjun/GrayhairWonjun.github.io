---
layout: post
title:  "[DRAFT] Spring Cloud OAuth2 with JWT"
comments: false
archive: false
date:   2018-02-21 22:47:00 -0500
categories: Java Spring
tags: java Spring oauth2 jwt
---

#OverView
Spring Cloud OAuth2를 활용하여 OAuth2 서버와 클라이언트를 만들 계획이다. 처음 만드는 서버는 메모리에 client와 user를 등록하여 동작하는 가장 간단한 형태로 개발을 할 예정이며 최초 버전이 완료되면 이후 JdbcClientDetailsService, JdbcUserDetailsService 를 개발하여 등록하여 DB에 등록된 Client와 User를 이용한 인증 시스템을 만드는 형태로 조금씩 복잡한 형태로 개발하여 실제 production 환경에서 사용할 수 있는 수준의 서버를 개발하는 것을 목표로 할 예정이다.

개발은 아래와 같이 진행할 예정이다.
# OAuth2 Server 개발
- Simple OAuth2 서버 개발
- Simple OAuth2 서버에 JWT를 access token 으로 사용
- 나만의 TokenStore, ApprovalStore, AuthorizationCodeService 를 만들어 토큰을 DB에 저장
- 나만의 UserDetailsService, ClientDetailsService를 만들어 DB에 Client, User 관리

# Resource Server 개발
- Spring Data Rest를 이용하여 Client, User 관리 서비스 개발
- Client Credentials Grant Type 으로 OAuth server 와 연동
- Authorization Code Grant Type 으로 OAuth Server 와 연동
- 

1. Spring Server