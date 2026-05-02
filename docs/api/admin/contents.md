# 관리자 컨텐츠 관리 API

관리자가 댓글 및 통계 데이터를 조회하고 삭제하는 API입니다.

---

## 1. 댓글 목록 조회

GET /admin/comments

### [설명]

전체 댓글 목록을 페이징으로 조회합니다.\
키워드 검색 시 댓글 내용 또는 작성자 닉네임에 대해 LIKE 검색을 수행합니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Query Parameters:
keyword  (선택) 검색 키워드 (댓글 내용 또는 닉네임)
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
        "content": "이 조합으로 유리",
        "likeCount": 5,
        "createdAt": "2026-02-28T10:30:00",
        "defenseCombo": "아멜리아,카구라,파이",
        "attackCombo": "아라곤,유신,카일"
      }
    ],
    "totalElements": 30,
    "totalPages": 2,
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
| id | Long | 댓글 ID |
| nickname | String | 작성자 닉네임 |
| content | String | 댓글 내용 |
| likeCount | int | 좋아요 수 |
| createdAt | LocalDateTime | 작성 일시 |
| defenseCombo | String | 방어 조합명 |
| attackCombo | String | 공격 조합명 |

---

## 2. 댓글 삭제

DELETE /admin/comments/{id}

### [설명]

특정 댓글을 삭제합니다. 관련 좋아요 데이터도 함께 삭제됩니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Path Parameters:
id  댓글 ID
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

## 3. 통계 목록 조회

GET /admin/stats

### [설명]

전체 통계 데이터를 페이징으로 조회합니다.\
키워드 검색 시 방어 조합 또는 공격 조합에 대해 LIKE 검색을 수행합니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Query Parameters:
keyword  (선택) 검색 키워드 (조합명)
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
        "defenseCombo": "아멜리아,카구라,파이",
        "defensePet": "유",
        "attackCombo": "아라곤,유신,카일",
        "attackPet": null,
        "totalGames": 150,
        "winRate": 62.5
      }
    ],
    "totalElements": 80,
    "totalPages": 4,
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
| id | Long | 통계 ID |
| defenseCombo | String | 방어 조합명 |
| defensePet | String | 방어 펫 (nullable) |
| attackCombo | String | 공격 조합명 |
| attackPet | String | 공격 펫 (nullable) |
| totalGames | int | 총 게임 수 |
| winRate | double | 승률 (%) |

---

## 4. 통계 삭제

DELETE /admin/stats/{id}

### [설명]

특정 통계를 삭제합니다. 관련 댓글과 좋아요 데이터도 함께 삭제됩니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Path Parameters:
id  통계 ID
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
| MATCHUP_COMMENT_NOT_FOUND | 404 | 존재하지 않는 댓글 ID |
| MATCHUP_STAT_NOT_FOUND | 404 | 존재하지 않는 통계 ID |

---

## [내부 처리 흐름]

```
JWT 인증 → ROLE_ADMIN 인가 → SuspensionGateFilter → AdminGateFilter(쿠키 검증)
    ↓
AdminController → AdminContentService
    ↓
getComments():    keyword 유무에 따라 전체/검색 조회 (Page 반환, createDate DESC)
deleteComment():  좋아요 벌크 삭제 → 댓글 삭제
getStats():       keyword 유무에 따라 전체/검색 조회 (Page 반환, totalGames DESC)
deleteStat():     좋아요 벌크 삭제 → 댓글 벌크 삭제 → 통계 삭제
```

### [삭제 시 cascade 순서]

```
통계 삭제:  좋아요(벌크) → 댓글(벌크) → 통계
댓글 삭제:  좋아요(벌크) → 댓글
```

벌크 삭제는 `@Modifying` + JPQL로 처리하여 N+1 문제를 방지합니다.
