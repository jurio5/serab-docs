# 방어 슬롯 목록 API

GET /rooms/{roomId}/castles/{castleId}/slots

특정 성의 방어 슬롯 목록을 조회하는 API입니다.

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
  "data": [
    {
      "id": 1,
      "position": 1,
      "playerName": "닉네임1",
      "formation": "PROTECT",
      "memo": "속공 200+ 조율자 필수",
      "heroes": ["델론스", "에이스", "레이첼"]
    },
    {
      "id": 2,
      "position": 2,
      "playerName": "닉네임2",
      "formation": "ATTACK",
      "memo": "파이 수문장 세팅 조심",
      "heroes": ["카일", "카구라", "파이]
    }
  ]
}
```

---

## [실패 응답]

가능한 ErrorCode:

- CASTLE_NOT_FOUND
- UNAUTHORIZED
