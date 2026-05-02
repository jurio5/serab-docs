# JWT 인증 필터 & 예외 처리 흐름

본 문서는  
**JwtAuthFilter**,  
**JwtExceptionFilter**,  
**Access Token Blacklist 처리**,  
**쿠키 기반 인증 흐름**을 설명합니다.

---

## 1. 인증 처리 흐름 (JwtAuthFilter)

요청이 들어오면 서버는 다음 순서로 인증을 진행합니다.

### 1) Access Token 쿠키 조회

```
access_token = CookieUtil.getCookie(request, "access_token")
```

### 2) Access Token 블랙리스트 확인

- logout 처리된 토큰인지 확인
- Redis: `BL:{accessToken}` 키 검사

### 3) Access Token 인증 시도

- JWT 파싱
- Claims → SecurityUser 변환
- 성공 시 SecurityContext 에 Authentication 저장

### 4) Access Token 만료 시 → Refresh Token 체크

- `JWT_EXPIRED` 발생 여부 확인
- Refresh Token 존재하면 **재발급 로직 진행**

### 5) 필터 제외 경로

다음 경로는 JwtAuthFilter를 거치지 않습니다:

| 경로 | 설명 |
|-----|------|
| `/ws/**` | WebSocket 연결 (STOMP) |

> WebSocket은 연결 시 별도의 인증 방식을 사용하거나, 연결 후 메시지 레벨에서 인증을 처리합니다.

---

## 2. Refresh Token 기반 재발급

Access Token 만료 후 다음 순서로 진행됩니다:

1. Refresh Token 쿠키 조회
2. JWT parse (type = refresh)
3. Redis에서 Refresh Token 유효성 확인
4. 새 Access Token + Refresh Token 생성
5. Redis에 새 Refresh Token 저장
6. 쿠키 업데이트
7. Authentication 다시 세팅

**→ Refresh Token Rotation 적용됨**

## 3. Refresh Token 재사용 감지 및 사용자 전체 Access Token 차단

Refresh Token 재사용 공격 등 보안 이벤트 발생 시
해당 사용자의 모든 Access Token을 무효화

Redis 구조
```java
BL_ALL:{memberId} = true
```

JwtAuthFilter에 다음 검사 로직을 추가

```java
if (memberId != null && accessTokenBlacklistService.isAllAccessBlacklisted(memberId)) {
log.warn("Access 전체 차단된 사용자 요청 차단. memberId={}", memberId);
filterChain.doFilter(request, response);
return;
}
```

Refresh Token 재사용 공격 감지 로직 추가

```java
public boolean isValid(Long memberId, String refreshToken) {
String saved = redis.opsForValue().get("REFRESH:" + memberId);

    if (saved == null) {
        return false;
    }

    if (refreshToken.equals(saved)) {
        return true;
    }

    onReuseDetected(memberId);
    return false;
}
```

재사용 공격 감지 시 처리:

```java
private void onReuseDetected(Long memberId) {
deleteRefreshToken(memberId);
accessTokenBlacklistService.blacklistAllAccess(memberId);

    log.warn("Refresh Token 재사용 공격 감지. memberId={}", memberId);
}
```

기존 Redis 구조에 아래 항목 추가

```java
BL_ALL:{memberId} = user all access blocked
```
---

## 4. 예외 처리 (JwtExceptionFilter)

JwtExceptionFilter는 **JwtAuthFilter보다 먼저 실행**되며  
모든 JWT 관련 예외를 통합적으로 JSON 형태로 반환합니다.

예외 예:

- 만료된 JWT
- 잘못된 Signature
- Refresh Token 불일치
- 위변조 의심

### 응답 형태 예시

```json
{
  "code": "JWT_EXPIRED",
  "message": "토큰이 만료되었습니다.",
  "data": null
}
```

## 5. 로그아웃 처리

일반 로그아웃 시:

- Refresh Token 삭제 (Redis key 삭제)
- Access Token → Blacklist 등록
- 아래 쿠키 삭제
    - access_token
    - refresh_token
    - role

Redis 구조:

```json
REFRESH:{memberId} = {refreshToken}
BL:{accessToken}   = logout
        
BL_ALL:{memberId}  = (Refresh Token 재사용 공격 감지 시에만 사용)
```
※ BL_ALL은 일반 로그아웃 시 사용되지 않으며  
Refresh Token 재사용 공격 등 보안 이벤트 발생 시에만 설정
---

## 6. 보안 요소 요약

- HttpOnly + Secure Cookie 기반 인증
- Refresh Token Rotation 적용
- Blacklist 기반 Access Token 폐기
- Redis TTL 기반 자동 만료 처리
- HMAC 검증된 OAuth Authorization Cookie

---

