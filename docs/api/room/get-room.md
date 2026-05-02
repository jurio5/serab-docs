# 방 조회 API

GET /rooms/{roomId}

특정 방의 상세 정보를 조회하는 API입니다.

---

## [설명]

방 ID를 통해 해당 방의 상세 정보를 조회합니다.

---

## [Request]

```
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
  "data": {
    "id": 1,
    "title": "1월 6일 길드전",
    "guildWarDate": "2026-01-06",
    "ownerNickname": "닉네임",
    "status": "ACTIVE",
    "memberCount": 5,
    "hasPassword": false,
    "waitingMode": true,
    "createdAt": "2026-01-06T23:55:00"
  }
}
```

---

## [실패 응답]

가능한 ErrorCode:

- ROOM_NOT_FOUND
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. roomId로 Room 조회
3. RoomResponse DTO 변환
4. 성공 응답 반환
