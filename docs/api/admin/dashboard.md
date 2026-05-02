# 관리자 대시보드 API

관리자 대시보드에 표시되는 통계 및 최근 활동 데이터를 조회하는 API입니다.

---

## 1. 대시보드 통계 조회

GET /admin/dashboard/stats

### [설명]

현재 서비스 상태를 요약한 4개 지표를 반환합니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...
```

별도 파라미터 없음

### [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": {
    "onlineUsers": 3,
    "activeRooms": 5,
    "todayBattleRecords": 12,
    "todayRoomsCreated": 2
  }
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| onlineUsers | int | 현재 WebSocket 접속 중인 사용자 수 |
| activeRooms | long | ACTIVE 상태인 방 수 |
| todayBattleRecords | long | 오늘 생성된 전투 기록 수 |
| todayRoomsCreated | long | 오늘 생성된 방 수 |

---

## 2. 최근 활동 조회

GET /admin/dashboard/activities

### [설명]

4가지 소스(방 생성, 전투 기록, 회원 가입, 댓글 작성)에서 최근 활동을 수집하여  
시간순(최신 우선)으로 최대 20건 반환합니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...
```

별도 파라미터 없음

### [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": [
    {
      "type": "BATTLE_RECORDED",
      "message": "[테스트 방] 아군 외성 1 — WIN",
      "createdAt": "2026-02-24T07:30:00"
    },
    {
      "type": "COMMENT_CREATED",
      "message": "테스터1님이 댓글 작성 (바네사+밀리아 vs 카일+카구라)",
      "createdAt": "2026-02-24T07:25:00"
    },
    {
      "type": "ROOM_CREATED",
      "message": "유저A님이 방 \"연습 방\" 생성",
      "createdAt": "2026-02-24T07:20:00"
    }
  ]
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| type | String | 활동 유형 (아래 표 참조) |
| message | String | 활동 요약 메시지 |
| createdAt | LocalDateTime | 활동 발생 시각 |

### 활동 유형

| type | 설명 | 메시지 형식 |
|------|------|------------|
| ROOM_CREATED | 방 생성 | `{닉네임}님이 방 "{제목}" 생성` |
| BATTLE_RECORDED | 전투 기록 | `[{방 제목}] {성 이름} — {결과}` |
| MEMBER_JOINED | 회원 가입 | `{닉네임}님이 가입` |
| COMMENT_CREATED | 댓글 작성 | `{닉네임}님이 댓글 작성 ({방어 조합} vs {공격 조합})` |

---

## [실패 응답]

| ErrorCode | HTTP Status | 설명 |
|-----------|-------------|------|
| UNAUTHORIZED | 401 | 로그인되지 않음 |
| ACCESS_DENIED | 403 | ROLE_ADMIN이 아님 |

---

## [내부 처리 흐름]

```
JWT 인증 → ROLE_ADMIN 인가 → AdminGateFilter(쿠키 검증)
    ↓
AdminController → AdminDashboardService
    ↓
getStats():           WebSocket 접속 수 + Repository count 쿼리 4건
getRecentActivities(): Repository 조회 4건 (각 10건) → 합산 → 정렬 → 상위 20건
```
