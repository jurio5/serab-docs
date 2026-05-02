# 전투 기록 삭제 API

DELETE /battle-records/{recordId}

전투 기록을 삭제하는 API입니다.

---

## [설명]

- 기록자 본인 또는 매니저 이상만 삭제 가능합니다.
- 삭제 시 복구할 수 없습니다.

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- recordId: 삭제할 기록 ID
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
- BATTLE_RECORD_EDIT_FORBIDDEN
