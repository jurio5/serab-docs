# 성 상세 조회 API

GET /rooms/{roomId}/castles/{castleId}

특정 성의 상세 정보를 조회하는 API입니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
- castleId: 성 ID
```
---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "id": 1,
    "side": "ALLY",
    "type": "OUTER",
    "number": 1,
    "displayName": "아군 외성 1",
    "memo": "닉네임1 배치",
    "defenderCount": 3,
    "maxDefenders": 10
  }
}
```

---

## [실패 응답]

가능한 ErrorCode:

- CASTLE_NOT_FOUND
- UNAUTHORIZED
