# 방어 슬롯 생성 API

POST /rooms/{roomId}/castles/{castleId}/slots

성에 새로운 방어 슬롯을 생성하는 API입니다.

---

## [설명]

지정된 성에 새로운 방어 슬롯을 생성합니다.\
플레이어 이름, 진형 정보를 설정할 수 있습니다.

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
- castleId: 성 ID

Body:
{
  "position": 1,
  "playerName": "닉네임1",
  "formation": "ATTACK",
  "memo": "속공 200+ 조율자 필수",
  "defenseSkillOrder": "파이2,카일2,카구라1",
  "heroes": ["카일", "카구라", "파이"]
}
```
---

## [진형 종류]

| 진형 | 설명 |
|------|------|
| DEFAULT | 기본 |
| BALANCE | 밸런스 |
| ATTACK | 공격 |
| PROTECT | 보호 |

---

## [비즈니스 정책]

| 조건 | 결과 |
|------|------|
| 위치가 최대 방어자 수 초과 | INVALID_SLOT_POSITION |
| 해당 위치에 이미 슬롯 존재 | SLOT_ALREADY_EXISTS |
| 플레이어 이름 없음 | PLAYER_NAME_INVALID |
| 진형 없음 | FORMATION_INVALID |

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "id": 1,
    "position": 1,
    "playerName": "닉네임1",
    "formation": "ATTACK",
    "memo": "속공 200+ 조율자 필수",
    "defenseSkillOrder": "파이2,카일2,카구라1",
    "heroes": ["카일", "카구라", "파이"]
  }
}
```

---

## [실패 응답]

가능한 ErrorCode:

- CASTLE_NOT_FOUND
- INVALID_SLOT_POSITION
- SLOT_ALREADY_EXISTS
- PLAYER_NAME_INVALID
- FORMATION_INVALID
- UNAUTHORIZED

---

## [실시간 이벤트]

방어 슬롯 생성 시 발행되는 WebSocket 이벤트:

| 이벤트 | 채널 | 설명 |
|-------|-----|-----|
| `DEFENDER_ADDED` | `/topic/room/{roomId}` | 새 방어팀 추가 알림 |
