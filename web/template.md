---
title: CSRF 공격
description:
author: codongmin
date: 2025-02-19T09:34:00
categories:
  - 웹
tags: ["csrf"]
image:
  path: /assets/img/thumbnail/.png
  lqip:
  alt: 이미지 설명
---

## CSRF 공격이란?

사이트간 요청 위조(Cross-site Request Forgery) 공격은 사용자가 자신의 의지와 상관없이 **공격자가 의도한 행위**를 **특정 사이트에 요청**하도록 하는 것을 의미.

예를 들어 특정 사용자가 A 서비스에서 로그인을 수행하고 서버는 해당 사용자에 대한 세션 ID를 `Set-Cookie` 헤더에 담아서 응답합니다. 그리고 클라이언트는 쿠키를 저장하고 요청마다 자동으로 전달합니다.

이러한 사용자를 대상으로 공격자는 악성 스크립트가 담긴 페이지에 접속하고록 유도.

- 악성 스크립트가 포함된 메일이나, 게시글,
- 악성 스크립트가 포함된 공격자 사이트 접속 링크를 전달하는 것이 대표적

사용자가 악성 스크립트가 포함된 페이지에 접속하게 되면 악성 스크립트가 실행됨,

이 스크립트는 사용자 의도와 상관없는 요청(결제, 비밀번호 변경)을 공격 대상 서버로 보내도록 구현되어 있음. 해당 요청은 브라우저에 의해서 자동으로 쿠기에 저장된 세션ID를 함께 전달함.

예를 들어, 공격자가 만든 사이트 내부에는 다음과 같은 태그가 존재할 수 있습니다.

```html
<img src="https://maeil-mail.com/member/changePassword?newValue=1234" />
```

공격자 사이트에 방문한 사용자는 자신의 의지와 무관하게 img 태그로 인해 세션 ID가 포함된 쿠키와 함께 비밀번호 변경 요청을 A 서버로 전달합니다.

## CSRF공격을 막는 방법.

교차 출처인 상황에서의 요청을 막는 방식으로 CSRF를 방어할 수 있음.

- 교차 출처란 현재 웹 페이지의 출처(origin)와 다른 출처를 가진 리소스에 접근하는 것을 의미 여기서 출처는 프로토콜, 도메인(호스트 이름), 포트의 조합을 말하며, 이 세 가지 요소 중 하나라도 다르면 다른 출처로 간주
  1. 프로토콜 (예: http, https)
  2. 도메인 (예: example.com)
  3. 포트 (예: 80, 443)

첫번째로 HTTP 헤더 중 하나인 `Referer` 요청 헤더를 사용하는 방법. `Referer` 요청 헤더 현재 요청을 보낸 페이지의 주소를 알 수 있음. 해당 주소와 Host(서버의 도메인 이름) 헤더를 비교하여 다른 경우, 예외를 발생시킬 수 있음.

- 하지만 Referer 요청 헤더는 조작될 수 있다는 점에서 한계가 존재함.

두번째로 CSRF 토큰을 사용.

페이지를 생성하기 이전에 사용자 세션에 임의의 CSRF 토큰을 저장. 그리고, 특정 API 요청에 대한 제출 폼을 생성할 때 해당 CSRF 토큰값이 설정된 input 태그를 추가.

```html
<input type="hidden" name="csrf_token" value="csrf_token_12341234" />
```

실제로 요청이 전달될 때, 해당 input 태그의 CSRF 토큰과 사용자 세션 내부에 존재하는 CSRF 토큰의 일치 여부를 판단하여, CSRF 공격에 대해 방어할 수 있음.

세번째로 SameSite 쿠키를 사용하여 크로스 사이트에 대한 쿠키 전송을 제어.

- 단 공격자가 같은 사이트 내에서 게시물을 통해 공격할 경우 무효화될 수 있음.

> NO Cookie == NO CSRF

## 참고 자료

- [CSRF(Cross-Site Request Forgery) Attack and Defence](https://junhyunny.github.io/information/security/spring-boot/spring-security/cross-site-reqeust-forgery/)
- [브라우저 쿠키와 SameSite 속성](https://seob.dev/posts/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EC%BF%A0%ED%82%A4%EC%99%80-SameSite-%EC%86%8D%EC%84%B1/)
