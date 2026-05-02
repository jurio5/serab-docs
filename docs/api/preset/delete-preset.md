# 프리셋 삭제 API

DELETE /presets/{presetId}

프리셋을 삭제하는 API입니다.

---

## [설명]

본인의 프리셋만 삭제할 수 있습니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- presetId: 프리셋 ID
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

- PRESET_NOT_FOUND
- PRESET_NOT_OWNER
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. 프리셋 조회
3. 소유자 검증
4. 프리셋 삭제
5. 성공 응답 반환
