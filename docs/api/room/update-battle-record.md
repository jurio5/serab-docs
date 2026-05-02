# 전투 기록 수정 API

PUT /battle-records/{recordId}

전투 기록을 수정하는 API입니다.

---

## [설명]

- 기록자 본인 또는 매니저 이상만 수정 가능합니다.
- 결과, 공격자 닉네임, 펫, 진형, 영웅 목록을 수정할 수 있습니다.
- 공격자 닉네임은 선택 사항입니다.

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- recordId: 기록 ID

Body:
{
  "result": "WIN",
  "attackerName": "공격자닉네임",
  "attackPetName": "유",
  "attackFormation": "ATTACK",
  "attackHeroes": ["델론즈", "에이스", "오공"]
}
```
---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "id": 1,
    "result": "WIN",
    "attackerName": "공격자닉네임",
    ...
  }
}
```

---

## [실패 응답]

가능한 ErrorCode:

- BATTLE_RECORD_NOT_FOUND
- BATTLE_RECORD_EDIT_FORBIDDEN
- ATTACK_HEROES_EMPTY
- ATTACK_HEROES_TOO_MANY
