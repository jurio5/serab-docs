# 격파 기록 조회 API

슬롯별, 방별 격파 기록을 조회하는 API입니다.

---

## [슬롯별 조회]

GET /battle-records/slots/{slotId}

특정 방어 슬롯의 모든 격파 기록을 조회합니다.

---

## [방별 조회]

GET /battle-records/rooms/{roomId}

특정 방의 모든 격파 기록을 조회합니다.

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": [
    {
      "id": 1,
      "defenderSlotId": 1,
      "defenderPlayerName": "방어자1",
      "defenseFormation": "PROTECT",
      "defenseHeroes": ["델론즈", "에이스", "파이"],
      "result": "WIN",
      "attackerName": "공격자1",
      "attackFormation": "ATTACK",
      "attackHeroes": ["카일", "카구라", "파이"],
      "recordedBy": "기록자1",
      "createdAt": "2026-01-06T23:55:00"
    }
  ]
}
```

---

## [실패 응답]

가능한 ErrorCode:

- DEFENDER_SLOT_NOT_FOUND
- ROOM_NOT_FOUND
- UNAUTHORIZED
