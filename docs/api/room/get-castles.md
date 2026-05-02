# 성 목록 조회 API

GET /rooms/{roomId}/castles

방의 모든 성 목록을 조회하는 API입니다.

---

## [설명]

지정된 방의 18개 성 목록을 조회합니다.\
각 성의 타입, 번호, 메모, 방어자 수 등을 확인할 수 있습니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
```
---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": [
    {
      "id": 1,
      "side": "ALLY",
      "type": "OUTER",
      "number": 1,
      "displayName": "아군 외성 1",
      "memo": "닉네임1 배치",
      "defenderCount": 3,
      "maxDefenders": 10
    },
    {
      "id": 10,
      "side": "ENEMY",
      "type": "OUTER",
      "number": 1,
      "displayName": "상대 외성 1",
      "memo": null,
      "defenderCount": 0,
      "maxDefenders": 10
    }
  ]
}
```

---

## [진영 종류]

| 진영 | 설명 |
|------|------|
| ALLY | 아군 |
| ENEMY | 상대 |

## [성 타입]

| 타입 | 설명 | 최대 방어자 |
|------|------|-----------|
| OUTER | 외성 | 10명 |
| INNER | 내성 | 15명 |
| MAIN | 본성 | 20명 |

---

## [실패 응답]

가능한 ErrorCode:

- ROOM_NOT_FOUND
- UNAUTHORIZED
