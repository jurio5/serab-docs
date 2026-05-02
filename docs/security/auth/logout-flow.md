# Logout 처리 구조

본 문서는 OAuth2 + JWT 기반 인증 시스템에서  
**로그아웃이 어떻게 처리되는지**  
구체적인 동작 흐름과 보안 요소를 설명합니다.

---

## 1. 로그아웃 목적

JWT 기반 시스템에서는 서버가 세션을 가지고 있지 않기 때문에  
“로그아웃”은 반드시 다음 두 가지를 처리해야 한다:

1. **Refresh Token 무효화**
    - Redis에 저장된 RT 삭제
2. **Access Token 사용 금지 처리**
    - Access Token을 Blacklist에 등록
    - 만료 전이라도 재사용 불가하도록 만듦

---

## 2. 로그아웃 전체 흐름

### 1) 클라이언트 → 서버로 로그아웃 요청

```
POST /api/v1/auth/logout
(with access_token + refresh_token 쿠키 포함)
```

---

### 2) Refresh Token 무효화

```java
refreshTokenService.deleteRefreshToken(memberId);
```

- Redis에서 해당 memberId의 Refresh Token 제거
- Refresh Token Rotation 구조를 사용하는 경우  
  → **탈취된 Refresh Token이더라도 즉시 무용지물이 됨**

---

### 3) Access Token Blacklist 등록

```java
accessTokenBlacklistService.register(accessToken);
```

- Redis에 `BL:{accessToken}` 형태로 저장
- TTL = Access Token 남은 시간  
  → 만료되면 자동 삭제됨
- Blacklist에 등록되면 곧바로 인증 불가

---

### 4) 인증 쿠키 삭제

```java
CookieUtil.deleteCookie(response, ACCESS_TOKEN_COOKIE);
CookieUtil.deleteCookie(response, REFRESH_TOKEN_COOKIE);
CookieUtil.deleteCookie(response, "role");
```

- 서버가 직접 쿠키의 만료시간을 0으로 덮어씀
- 클라이언트에서 인증 관련 모든 쿠키가 제거됨

---

## 3. 로그아웃 후 요청 처리

### Access Token이 이미 Blacklist에 있으면?

- JwtAuthFilter에서 다음 조건으로 인증 차단됨:

```java
if (accessTokenBlacklistService.isBlacklisted(accessToken)) {
    // 인증되지 않은 요청으로 처리
}
```

Blacklist에 올라간 토큰은  
**만료 시간까지 절대 인증 불가.**

---

## 4. 보안 효과

### Refresh Token 제거
→ 재발급 경로가 완전히 차단됨  
→ 탈취된 Refresh Token으로 Access Token 재발급 불가능

### Access Token Blacklist
→ 이미 클라이언트에게 발행된 토큰이라도 재사용 불가  
→ 즉시 무효화 효과

### 쿠키 삭제
→ 브라우저 내부에 남아있던 정보도 제거

### 완전한 Stateless 로직 유지
→ 서버가 세션을 가지지 않으면서도  
인증 무효화를 안정적으로 처리할 수 있음

---

## 5. 요약

**로그아웃 = (Refresh Token 삭제) + (Access Token Blacklist 등록) + (쿠키 삭제)**

이 세 가지가 모두 수행되면  
JWT 기반 시스템에서도 완전한 로그아웃이 가능

---
