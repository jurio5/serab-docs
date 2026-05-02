# 관리자 방 관리 API

관리자가 길드전 방 목록을 조회하고, 강제 종료 및 옵저버 입장을 수행하는 API입니다.

---

## 1. 방 목록 조회

GET /admin/rooms

### [설명]

상태별 방 목록을 페이징으로 조회합니다.\
키워드 검색 시 방 제목 또는 방장 닉네임에 대해 LIKE 검색을 수행합니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Query Parameters:
status   (선택, 기본값: ACTIVE) 방 상태 (ACTIVE, CLOSED, LOCKED)
keyword  (선택) 검색 키워드 (방 제목 또는 방장 닉네임)
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
        "title": "길드전 2월 25일",
        "ownerNickname": "방장유저",
        "memberCount": 5,
        "status": "ACTIVE",
        "guildWarDate": "2026-02-25",
        "hasPassword": false,
        "createdAt": "2026-02-24T20:00:00"
      }
    ],
    "totalElements": 10,
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
| id | Long | 방 ID |
| title | String | 방 제목 |
| ownerNickname | String | 방장 닉네임 |
| memberCount | int | 활성 참여자 수 (옵저버 제외) |
| status | String | 방 상태 (ACTIVE, CLOSED, LOCKED) |
| guildWarDate | LocalDate | 길드전 날짜 |
| hasPassword | boolean | 비밀번호 설정 여부 |
| createdAt | LocalDateTime | 생성 일시 |

---

## 2. 방 강제 종료

POST /admin/rooms/{id}/close

### [설명]

활성 상태의 방을 강제로 종료합니다.\
대기 중인 BattleRecord(PENDING)가 있으면 함께 삭제됩니다.\
방 안의 참여자와 로비에 `ROOM_CLOSED` WebSocket 이벤트가 발행됩니다.

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Path Parameters:
id  방 ID
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

## 3. 옵저버 입장

POST /admin/rooms/{id}/observe

### [설명]

관리자가 방에 옵저버 역할로 입장합니다.\
옵저버는 참여자 수에 포함되지 않으며, 방장 양도 후보에서도 제외됩니다.\
채팅은 가능하며 옵저버 전용 색상으로 표시됩니다.\
이미 방에 참여 중이면 무시됩니다 (중복 입장 방지).

### [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Path Parameters:
id  방 ID
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
| ROOM_NOT_FOUND | 404 | 존재하지 않는 방 ID |
| ROOM_ALREADY_CLOSED | 400 | 이미 종료된 방 |

---

## [내부 처리 흐름]

```
JWT 인증 → ROLE_ADMIN 인가 → SuspensionGateFilter → AdminGateFilter(쿠키 검증)
    ↓
AdminController → AdminRoomService
    ↓
getRooms():       status + keyword 조건으로 Page 조회 → AdminRoomResponse 변환
closeRoom():      PENDING BattleRecord 삭제 → room.close() → ROOM_CLOSED 이벤트 발행 (방 + 로비)
joinAsObserver(): 중복 체크 → RoomMember(OBSERVER) 저장 (이벤트 없음, 카운팅 제외)
```

### 옵저버 동작 규칙

| 항목 | 동작 |
|------|------|
| 참여자 목록 | 미표시 |
| 카운팅 (로비/방) | 제외 |
| 방장 양도 후보 | 제외 |
| 채팅 | 가능 (OBSERVER 색상) |
| 퇴장 시 이벤트 | 없음 (조용히 삭제) |
| 일반 재입장 | OBSERVER → USER/OWNER 역할 전환 |
