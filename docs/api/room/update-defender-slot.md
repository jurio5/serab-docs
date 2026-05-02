# 방어 슬롯 수정 API

PUT /rooms/{roomId}/castles/{castleId}/slots/{slotId}

방어 슬롯의 정보를 수정하는 API입니다.

---

## [설명]

방어 슬롯의 플레이어 이름, 진형, 영웅 목록, 메모를 수정할 수 있습니다.\
모든 필드는 선택적으로 수정 가능합니다.

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
- castleId: 성 ID
- slotId: 슬롯 ID

Body:
{
  "playerName": "닉네임1",
  "formation": "PROTECT",
  "memo": "속공 200+ 조율자 필수",
  "heroes": ["에이스", "레이첼", "루디"]
}
```
---

## [성공 응답]

```json
HTTP 200
{
  "success": true
}
```

---

## [실패 응답]

가능한 ErrorCode:

- DEFENDER_SLOT_NOT_FOUND
- UNAUTHORIZED

---

## [실시간 이벤트]

방어 슬롯 수정 시 발행되는 WebSocket 이벤트:

| 이벤트 | 채널 | 설명 |
|-------|-----|-----|
| `DEFENDER_UPDATED` | `/topic/room/{roomId}` | 방어팀 수정 알림 |
