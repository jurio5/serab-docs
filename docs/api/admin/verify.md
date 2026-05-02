# 관리자 비밀번호 검증 API

POST /admin/verify

관리자 페이지 접근을 위한 2단계 비밀번호 검증 API입니다.

---

## [설명]

ROLE_ADMIN 인증 후 추가 비밀번호 검증을 수행합니다.  
검증 성공 시 HMAC 서명된 HttpOnly 쿠키(admin_gate)를 발급하며,  
이후 `/admin/**` 요청은 이 쿠키로 접근 권한을 확인합니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Body (JSON):
{
  "password": "관리자_비밀번호"
}
```

### 요청 필드

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| password | String | ✅ | 관리자 입장 비밀번호 |

---

## [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": null
}
```

응답 헤더에 `Set-Cookie: admin_gate=...` 가 포함됩니다.

### 쿠키 속성

| 속성 | 값 |
|------|------|
| HttpOnly | true |
| MaxAge | 3600 (1시간) |
| SameSite | Lax |

---

## [실패 응답]

| ErrorCode | HTTP Status | 설명 |
|-----------|-------------|------|
| ADMIN_PASSWORD_INCORRECT | 403 | 비밀번호 불일치 |
| UNAUTHORIZED | 401 | 로그인되지 않음 |
| ACCESS_DENIED | 403 | ROLE_ADMIN이 아님 |

---

## [내부 처리 흐름]

1. SecurityFilterChain에서 ROLE_ADMIN 확인 (인가)
2. AdminController → AdminService.verifyAndIssueGateCookie() 위임
3. bcrypt로 비밀번호 검증 (`PasswordEncoder.matches()`)
4. 검증 성공 → HMAC 서명 gate 쿠키 발급
5. 검증 실패 → ADMIN_PASSWORD_INCORRECT 예외

---

## [보안 구조]

```
JWT 인증 → ROLE_ADMIN 인가 → 비밀번호 검증 → HMAC 쿠키 발급
                                                   ↓
                            이후 /admin/** 요청 → AdminGateFilter가 쿠키 검증
```
