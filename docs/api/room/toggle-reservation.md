# 공격 예약 토글 API

POST /rooms/{roomId}/castles/{castleId}/slots/{slotId}/reserve

방어 슬롯에 대한 공격 예약을 토글하는 API입니다.

---

## [설명]

방어 슬롯에 공격 예약을 등록하거나 취소합니다.\
본인이 이미 예약 중이면 취소, 아니면 예약 등록됩니다.\
다른 사용자가 예약 중이어도 덮어쓰기가 가능합니다 (프론트에서 확인 후 요청).

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
- castleId: 성 ID
- slotId: 방어 슬롯 ID

Body: 없음
```

---

## [동작 방식]

| 현재 상태 | 요청자 | 결과 |
|---------|-------|------|
| 예약 없음 | A | A가 예약자로 등록 |
| A가 예약 중 | A | 예약 취소 (null) |
| B가 예약 중 | A | A가 예약자로 변경 |

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "id": 1,
    "castleId": 1,
    "position": 1,
    "playerName": "닉네임1",
    "formation": "ATTACK",
    "memo": "메모",
    "petName": "유",
    "result": null,
    "heroes": [...],
    "modifiedByNickname": "기록자1",
    "reservedByMemberId": 123,
    "reservedByNickname": "예약자닉네임"
  }
}
```

---

## [실패 응답]

가능한 ErrorCode:

- DEFENDER_SLOT_NOT_FOUND
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. DefenderSlot 조회
3. 현재 사용자 확인 (RequestContext.getActor())
4. **예약 상태에 따른 분기:**
   - 본인 예약 중 → `cancelReservation()`
   - 그 외 → `reserve(memberId, nickname)`
5. WebSocket 이벤트 발행 (`DEFENDER_UPDATED`)
6. 성공 응답 반환
