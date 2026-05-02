# 프리셋 상세 조회 API

GET /presets/{presetId}

프리셋의 상세 데이터를 조회하는 API입니다.

---

## [설명]

본인의 프리셋만 상세 조회할 수 있습니다.\
상세 조회 시 `data` 필드(직렬화된 배치 데이터)가 포함됩니다.

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
  "success": true,
  "data": {
    "id": 1,
    "name": "프리셋 이름",
    "data": "[{\"type\":\"KEEP\",\"number\":1,...}]",
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
4. 상세 응답 반환 (data 포함)
