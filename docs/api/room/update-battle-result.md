# 격파 결과 수정 API

PATCH /battle-records/{recordId}/result

격파 기록의 결과(승/패)를 수정하는 API입니다.

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- recordId: 격파 기록 ID

Body:
{
  "result": "LOSE"
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

- BATTLE_RECORD_NOT_FOUND
- BATTLE_RESULT_INVALID
- UNAUTHORIZED
