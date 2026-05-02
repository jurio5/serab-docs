# 격파 기록 생성 API

POST /battle-records/slots/{slotId}

방어 슬롯에 대한 격파 기록을 생성하는 API입니다.

---

## [설명]

특정 방어 슬롯을 공격한 결과를 기록합니다.\
공격 진형, 공격 영웅, 승/패 결과를 저장합니다.

**WIN/LOSE 처리 정책:**
- **WIN**: 슬롯당 1개만 등록 가능. 이미 존재하면 `409 CONFLICT` 에러 반환 (수정은 수정 API 이용)
- **LOSE**: 항상 INSERT (공격 시도 히스토리 누적, 여러 개 저장 가능)

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- slotId: 방어 슬롯 ID

Body:
{
  "result": "WIN",
  "attackerName": "닉네임2",
  "attackFormation": "ATTACK",
  "attackHeroes": ["카일", "카구라", "파이"],
  "attackSkillOrder": "카일2,카구라1,파이2"
}
```
---

## [결과 종류]

| 결과 | 설명 | 저장 방식 |
|------|------|----------|
| WIN | 승리 | INSERT (1개만 허용, 중복 시 409) |
| LOSE | 패배 | 항상 INSERT (히스토리 누적) |

---

## [비즈니스 정책]

| 조건          | 결과 |
|-------------|------|
| 결과 없음       | BATTLE_RESULT_INVALID |
| 공격자 이름 없음   | ATTACKER_NAME_INVALID |
| 진형 없음       | FORMATION_INVALID |
| 공격 영웅 없음    | ATTACK_HEROES_EMPTY |
| 공격 영웅 3명 초과 | ATTACK_HEROES_TOO_MANY |
| WIN 기록 중복   | WIN_RECORD_ALREADY_EXISTS (409) |

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "id": 1,
    "defenderSlotId": 1,
    "defenderPlayerName": "방어자1",
    "defenseFormation": "PROTECT",
    "defenseHeroes": ["델론즈", "에이스", "파이"],
    "defenseSkillOrder": "파이2,카일2,카구라1",
    "result": "WIN",
    "attackerName": "공격자1",
    "attackFormation": "ATTACK",
    "attackHeroes": ["카일", "카구라", "파이"],
    "attackSkillOrder": "카일2,카구라1,파이2",
    "recordedBy": "기록자1",
    "createdAt": "2026-01-06T23:55:00"
  }
}
```

---

## [실패 응답]

가능한 ErrorCode:

- DEFENDER_SLOT_NOT_FOUND
- BATTLE_RESULT_INVALID
- FORMATION_INVALID
- ATTACK_HEROES_EMPTY
- ATTACK_HEROES_TOO_MANY
- WIN_RECORD_ALREADY_EXISTS
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. CreateBattleRecord VO 생성 → 검증
3. DefenderSlot 조회
4. **WIN 중복 검증**: 해당 슬롯에 WIN 기록이 이미 존재하면 `WIN_RECORD_ALREADY_EXISTS` 에러 반환
5. BattleRecord INSERT (attackSkillOrder, defenseSkillOrder 포함)
6. DefenderSlot.result 업데이트
7. **스킬 순서 통계 업데이트**: MatchupSkillStat에 해당 스킬 조합 기록 누적 (배치 처리)
8. WebSocket 이벤트 발행 (`DEFENDER_UPDATED`, `BATTLE_RECORDED`)
9. 성공 응답 반환
