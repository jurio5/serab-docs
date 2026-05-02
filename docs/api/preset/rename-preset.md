# 프리셋 이름 변경 API

PATCH /presets/{presetId}/name

프리셋의 이름을 변경하는 API입니다.

---

## [설명]

본인의 프리셋 이름만 변경할 수 있습니다.

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- presetId: 프리셋 ID

Body:
{
  "name": "새 프리셋 이름"
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
    "name": "새 프리셋 이름",
    "data": null,
    "createdAt": "2026-02-22T06:00:00"
  }
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
4. 이름 변경
5. 요약 응답 반환
