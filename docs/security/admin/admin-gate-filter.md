# Admin Gate Filter 구조

관리자 페이지에 대한 2단계 보안 게이트 필터 구조를 설명합니다.

---

## 1. 필터 체인에서의 위치

```
요청 →
JwtExceptionFilter →
JwtAuthFilter →
SuspensionGateFilter →        ← 정지 유저 쓰기 차단
AdminGateFilter →              ← /admin/** 요청에만 동작
UsernamePasswordAuthenticationFilter →
Spring Security 인증/인가 처리
```

JwtAuthFilter 이후에 실행되므로,  
이미 JWT 인증과 ROLE_ADMIN 인가를 통과한 요청만 도달합니다.

---

## 2. 동작 조건

| 요청 경로 | 필터 동작 여부 |
|-----------|------------|
| `/admin/verify` | 제외 (쿠키 발급 엔드포인트) |
| `/admin/**` | 동작 |
| 나머지 (`/rooms`, `/members` 등) | 무시 |

---

## 3. 쿠키 검증 흐름

```
1. 요청에서 admin_gate 쿠키 추출
2. 쿠키 없음 → 403 (ADMIN_GATE_REQUIRED)
3. 쿠키 값 파싱: base64Payload.signature
4. Base64 디코딩 → payload: "admin-verified:{timestamp}"
5. prefix 검증: "admin-verified:" 로 시작하는지 확인
6. 타임스탬프 검증: 발급 후 1시간 이내인지 확인
7. HMAC 서명 검증: HmacUtil.sign(payload) 와 signature 비교
8. 모든 검증 통과 → filterChain.doFilter() (요청 진행)
```

---

## 4. HMAC 서명 구조

```
쿠키 값: base64(admin-verified:1740000000).HmacSHA256서명

payload (Base64 디코딩 후):
  "admin-verified:1740000000"
   ├── prefix: "admin-verified:" → 쿠키 용도 식별
   └── timestamp: 1740000000    → 발급 시각 (밀리초)

signature:
  HmacSHA256(payload, serverSecret) → 서버만 생성/검증 가능
```

---

## 5. 보안 설계 요약

- HMAC 서명으로 쿠키 위조 불가
- HttpOnly 쿠키로 XSS 공격 방지
- 1시간 만료로 세션 하이재킹 제한
- 서버 비밀키 없이는 유효한 쿠키 생성 불가
- bcrypt 해싱으로 설정 파일 유출 시에도 원문 비밀번호 보호

---
