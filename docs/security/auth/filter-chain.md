# Security Filter Chain 구조

본 문서는 Spring Security 기반 API 서버에서  
OAuth2 + JWT 인증이 어떻게 처리되는지  
**필터 체인 전체 흐름**을 설명합니다.

---

## 1. 전체 요청 흐름 요약

요청 →  
**JwtExceptionFilter** →  
**JwtAuthFilter** →  
(UsernamePasswordAuthenticationFilter) →  
Spring Security 인증/인가 처리

---

## 2. Custom Filter 적용 순서

### ① JwtExceptionFilter
- **가장 먼저 실행됨**
- 인증 과정에서 발생하는 모든 JWT 관련 예외를 캐치
- JSON 응답으로 변환

### ② JwtAuthFilter
- 쿠키에서 Access Token / Refresh Token 읽기
- Access Token 인증
- Access Token 만료 시 Refresh Token 기반 재발급 진행
- SecurityContext 에 Authentication 저장

### ③ UsernamePasswordAuthenticationFilter
- 기본 Spring Security 로그인 처리
- 이 프로젝트에서는 사용하지 않지만 기준점이므로 이 앞에 필터를 추가함

---

## 3. 인증 성공 후 SecurityContext

JwtAuthFilter는 Access Token 검증 후:

```
SecurityContextHolder.getContext().setAuthentication(auth);
```

요청이 끝나면 항상:

```
finally {
    SecurityContextHolder.clearContext();
}
```

Stateless 구조 보장을 위해 요청 단위로 인증 정보를 제거한다.

---

## 4. OAuth2 Login 필터 흐름

OAuth2 인증이 포함될 경우:

1. OAuth2AuthorizationRequestRedirectFilter
2. OAuth2LoginAuthenticationFilter
3. **CustomOAuth2UserService**
4. **CustomOAuth2AuthenticationSuccessHandler**
5. 쿠키에 Access/Refresh Token 저장
6. Redirect 완료

JWT 필터와 OAuth2 필터는 서로 충돌하지 않으며,  
OAuth2 로그인 성공 후 생성된 JWT가 이후 인증을 담당한다.

---

## 5. SecurityConfig 내 필터 등록 위치

```java
.addFilterBefore(new JwtExceptionFilter(), JwtAuthFilter.class)
.addFilterBefore(new JwtAuthFilter(...), UsernamePasswordAuthenticationFilter.class)
```

의미:

- JwtExceptionFilter → JwtAuthFilter 보다 먼저 실행
- JwtAuthFilter → UsernamePasswordAuthenticationFilter 보다 먼저 실행

---

## 6. Stateless 구조 정리

- 서버는 세션을 사용하지 않음  
  → `SessionCreationPolicy.STATELESS`
- 모든 요청은 쿠키의 JWT로 인증
- 서버는 인증 상태를 저장하지 않음
- Redis는 Refresh Token / Blacklist만 관리

---

