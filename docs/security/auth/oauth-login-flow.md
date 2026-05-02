# OAuth2 로그인 처리 문서

이 문서는 OAuth2 로그인 흐름, 신규/기존 회원 처리, JWT 발급, 쿠키 저장 방식, 실패 리다이렉트 규칙을 정리합니다.

## 1. 개요

지원 Provider:

- Kakao
- Google

로그인 흐름:

1. 클라이언트가 `/oauth2/authorization/{provider}` 호출
2. 서버가 OAuth Authorization Request를 쿠키에 저장
3. Provider 인증 페이지로 리다이렉트
4. 인증 성공 시 `/login/oauth2/code/{provider}` 콜백 도착
5. 서버가 사용자 정보를 조회하고 신규/기존 회원 여부를 판별
6. JWT 발급 또는 회원가입 유도
7. 프론트로 `status` 파라미터와 함께 리다이렉트

## 2. Authorization Request 저장 방식

OAuth 로그인 시작 시 서버는 아래 정보를 쿠키에 저장합니다.

- Authorization Request
- Redirect URI

보안 정책:

- Base64 URL Encoding
- HMAC-SHA256 서명 검증
- 위변조 또는 역직렬화 실패 시 인증 중단

## 3. 성공 처리

### 신규 사용자

처리:

1. `provider`, `oauthId`, `email`을 임시 HttpOnly Secure 쿠키로 저장
2. 프론트로 `status=REGISTER` 리다이렉트
3. 프론트는 회원가입 화면으로 이동

### 기존 사용자

처리:

1. Access Token / Refresh Token 발급
2. Refresh Token Redis 저장
3. 인증 관련 쿠키 저장
4. Authorization Request 쿠키 삭제
5. 프론트로 `status=SUCCESS` 리다이렉트

## 4. 실패 처리

OAuth 실패는 JSON 에러 페이지를 직접 반환하지 않고, 프론트로 리다이렉트합니다.

### 실패 상태값

- `OAUTH_EXPIRED`
  - `authorization_request_not_found`
  - 로그인 요청 쿠키가 만료되었거나 이미 소비된 콜백이 다시 들어온 경우
- `OAUTH_LOGIN_FAILED`
  - 그 외 일반 OAuth 로그인 실패

### 실패 시 동작

1. 실패 핸들러가 예외 코드를 판별
2. 프론트 기본 URL로 `?status=...`를 붙여 리다이렉트
3. 프론트는 상태값에 따라 안내 메시지 표시

예시:

```text
https://serab.dev?status=OAUTH_EXPIRED
https://serab.dev?status=OAUTH_LOGIN_FAILED
```

## 5. 프론트에서 사용하는 상태값

- `REGISTER`: 신규 OAuth 사용자, 회원가입 화면 이동
- `SUCCESS`: 기존 사용자 로그인 성공
- `RESTORED`: 탈퇴 회원 자동 복구 성공
- `RESTORE_FAILED`: 탈퇴 회원 복구 실패
- `OAUTH_EXPIRED`: 로그인 요청 만료 또는 중복 콜백
- `OAUTH_LOGIN_FAILED`: 일반 OAuth 로그인 실패

## 6. 쿠키 정책

### Access Token

- HttpOnly = true
- Secure = true
- SameSite = Lax

### Refresh Token

- HttpOnly = true
- Secure = true
- Redis 저장

### role 쿠키

- HttpOnly = false
- 프론트 UI 분기 용도

## 7. 요약

1. 로그인 시작
2. Authorization Request 쿠키 저장
3. Provider 인증
4. 사용자 정보 조회
5. 신규/기존 사용자 분기
6. 성공 시 JWT 발급 후 프론트 리다이렉트
7. 실패 시 상태값과 함께 프론트 리다이렉트
