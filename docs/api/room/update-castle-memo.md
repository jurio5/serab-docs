# 성 메모 수정 API

PATCH /rooms/{roomId}/castles/{castleId}/memo

성의 메모를 수정하는 API입니다.

---

## [설명]

성에 메모를 추가하거나 수정합니다.\
방어팀 배치 정보 등을 기록하는 용도입니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
- castleId: 성 ID

Body:
"닉네임1 배치, 영웅 조합, 정보 등"
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

- CASTLE_NOT_FOUND
- UNAUTHORIZED

---

## [실시간 이벤트]

성 메모 수정 시 발행되는 WebSocket 이벤트:

| 이벤트 | 채널 | 설명 |
|-------|-----|-----|
| `CASTLE_UPDATED` | `/topic/room/{roomId}` | 성 정보 변경 알림 |
