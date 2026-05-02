# 관리자 회원 관리 API

관리자가 회원 목록을 조회하고, 활동 정지/해제를 수행하는 API입니다.

---

## 1. 회원 목록 조회

GET /admin/members

### [설명]

전체 회원 목록을 페이징으로 조회합니다.\
키워드 검색 시 닉네임 또는 이메일에 대해 LIKE 검색을 수행합니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Query Parameters:
keyword  (선택) 검색 키워드 (닉네임 또는 이메일)
page     (선택, 기본값: 0)
size     (선택, 기본값: 20)
```

### [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": {
    "content": [
      {
        "id": 1,
        "nickname": "유저A",
        "email": "user@example.com",
        "role": "ROLE_USER",
        "createdAt": "2026-01-15T10:30:00",
        "suspendedUntil": null,
        "suspendReason": null
      }
    ],
    "totalElements": 150,
    "totalPages": 8,
    "number": 0,
    "size": 20,
    "first": true,
    "last": false
  }
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 회원 ID |
| nickname | String | 닉네임 |
| email | String | 이메일 |
| role | String | 권한 (ROLE_USER, ROLE_ADMIN) |
| createdAt | LocalDateTime | 가입 일시 |
| suspendedUntil | LocalDateTime | 정지 만료 일시 (null이면 정상) |
| suspendReason | String | 정지 사유 (null이면 정상) |

---

## 2. 탈퇴 회원 목록 조회

GET /admin/members/deleted

### [설명]

탈퇴한 회원 목록을 페이징으로 조회합니다. 키워드 검색 시 닉네임 또는 이메일에 대해 LIKE 검색을 수행합니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Query Parameters:
keyword  (선택) 검색 키워드 (닉네임 또는 이메일)
page     (선택, 기본값: 0)
size     (선택, 기본값: 20)
```

### [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": {
    "content": [
      {
        "id": 1,
        "nickname": "탈퇴유저",
        "email": "deleted@example.com",
        "deletedAt": "2026-02-20T15:00:00",
        "expireAt": "2026-03-22T15:00:00"
      }
    ],
    "totalElements": 5,
    "totalPages": 1,
    "number": 0,
    "size": 20,
    "first": true,
    "last": true
  }
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 탈퇴 회원 ID |
| nickname | String | 닉네임 |
| email | String | 이메일 |
| deletedAt | LocalDateTime | 탈퇴 일시 |
| expireAt | LocalDateTime | 데이터 만료(삭제 예정) 일시 |

---

## 3. 활동 정지

POST /admin/members/{id}/suspend

### [설명]

특정 회원의 활동을 정지합니다. `days=0`이면 영구 정지로 처리됩니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Path Parameters:
id  회원 ID

Body (JSON):
{
  "days": 7,
  "reason": "부적절한 댓글"
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| days | int | 정지 기간 (일 단위, 0 = 영구 정지) |
| reason | String | 정지 사유 |

### [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": null
}
```

---

## 4. 활동 정지 해제

DELETE /admin/members/{id}/suspend

### [설명]

특정 회원의 활동 정지를 즉시 해제합니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Path Parameters:
id  회원 ID
```

### [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": null
}
```

---

## [실패 응답]

| ErrorCode | HTTP Status | 설명 |
|-----------|-------------|------|
| UNAUTHORIZED | 401 | 로그인되지 않음 |
| ACCESS_DENIED | 403 | ROLE_ADMIN이 아님 |
| ADMIN_GATE_REQUIRED | 403 | admin_gate 쿠키 없음 |
| MEMBER_NOT_FOUND | 404 | 존재하지 않는 회원 ID |

---

## [내부 처리 흐름]

```
JWT 인증 → ROLE_ADMIN 인가 → SuspensionGateFilter → AdminGateFilter(쿠키 검증)
    ↓
AdminController → AdminMemberService
    ↓
getMembers():       keyword 유무에 따라 전체/검색 조회 (Page 반환)
suspendMember():    Member.suspend(until, reason) → 더티 체킹으로 DB 반영
unsuspendMember():  Member.unsuspend() → suspendedUntil/reason null 처리
```
