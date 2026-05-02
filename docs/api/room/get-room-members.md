# 방 멤버 목록 조회 API

GET /rooms/{roomId}/members

방에 참여 중인 멤버 목록을 조회하는 API입니다.

---

## [설명]

해당 방에 참여 중인 모든 멤버 정보를 조회합니다.
`ACTIVE`와 `OFFLINE` 멤버를 모두 포함하며,
`KICKED` 또는 `OBSERVER` 멤버는 제외됩니다.
역할(OWNER, MANAGER, USER) 순으로 정렬되어 반환됩니다.

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 조회할 방 ID
```

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": [
    {
      "id": 11,
      "memberId": 1,
      "nickname": "방장닉네임",
      "role": "OWNER",
      "status": "ACTIVE",
      "isCurrentUser": false
    },
    {
      "id": 12,
      "memberId": 2,
      "nickname": "오프라인닉네임",
      "role": "USER",
      "status": "OFFLINE",
      "isCurrentUser": false
    },
    {
      "id": 13,
      "memberId": 3,
      "nickname": "내닉네임",
      "role": "USER",
      "status": "ACTIVE",
      "isCurrentUser": true
    }
  ]
}
```

---

## [응답 필드 설명]

| 필드 | 타입 | 설명 |
|-----|-----|-----|
| id | Long | RoomMember ID |
| memberId | Long | 멤버 ID |
| nickname | String | 닉네임 |
| role | String | 역할 (OWNER/MANAGER/USER) |
| status | String | 참여 상태 (`ACTIVE` / `OFFLINE`) |
| isCurrentUser | Boolean | 현재 로그인 사용자 여부 |

프론트 참여자 패널에서는 `status = OFFLINE`인 멤버를
역할 대신 `오프라인`으로 표시합니다.

---

## [실패 응답]

가능한 ErrorCode:

- ROOM_NOT_FOUND
- UNAUTHORIZED

---

## [실시간 동기화]

이 API와 관련된 WebSocket 이벤트:

| 이벤트 | 발생 시점 |
|-------|---------|
| MEMBER_JOINED | 새 멤버 입장 시 |
| MEMBER_LEFT | 멤버 퇴장 시 |
| MEMBER_ROLE_CHANGED | 역할 변경 시 |
| MEMBER_STATUS_CHANGED | `ACTIVE / OFFLINE` 상태 전환 시 |

구독 채널: `/topic/room/{roomId}`
