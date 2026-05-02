# 내 프로필 조회 API

GET /members/me

로그인된 사용자의 프로필 정보를 조회하는 API 입니다.

---

## [설명]

Access Token 기반으로 현재 로그인된 사용자의 프로필을 반환합니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...
```

별도의 Body 없이 인증 쿠키만 필요합니다.

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "id": 1,
    "nickname": "유저닉네임",
    "email": "user@example.com",
    "role": "ROLE_USER",
    "createdAt": "2026-01-15T12:00:00",
    "nicknameChangedAt": "2026-02-01T09:30:00"
  }
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 회원 ID |
| nickname | String | 현재 닉네임 |
| email | String | 이메일 |
| role | String | 권한 (ROLE_USER, ROLE_ADMIN) |
| createdAt | DateTime | 가입 일시 |
| nicknameChangedAt | DateTime | 마지막 닉네임 변경 일시 (nullable) |

---

## [실패 응답]

| ErrorCode | 설명 |
|-----------|------|
| UNAUTHORIZED | 인증되지 않은 사용자 |

---

## [내부 처리 흐름]

1. 인증 쿠키에서 사용자 정보 추출
2. RequestContext에서 현재 사용자(Actor) 조회
3. Member → MemberResponse 변환 후 반환
