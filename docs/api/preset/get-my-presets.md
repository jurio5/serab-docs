# 내 프리셋 목록 조회 API

GET /presets

내가 저장한 프리셋 목록을 조회하는 API입니다.

---

## [설명]

로그인한 사용자의 프리셋 목록을 생성일 기준 내림차순으로 반환합니다.\
목록 조회 시 `data` 필드는 포함되지 않습니다 (요약 응답).

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...
```
---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": [
    {
      "id": 3,
      "name": "프리셋3",
      "data": null,
      "createdAt": "2026-02-22T06:00:00"
    },
    {
      "id": 2,
      "name": "프리셋2",
      "data": null,
      "createdAt": "2026-02-21T15:00:00"
    }
  ]
}
```

---

## [실패 응답]

가능한 ErrorCode:

- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. 로그인 사용자의 프리셋 조회 (생성일 내림차순)
3. 요약 응답 변환 (data 제외)
4. 성공 응답 반환
