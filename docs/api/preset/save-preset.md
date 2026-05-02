# 프리셋 저장 API

POST /presets?roomId={roomId}

현재 방의 아군 진영 배치를 프리셋으로 저장하는 API입니다.

---

## [설명]

현재 방의 아군(ALLY) 성들의 방어 배치를 JSON으로 직렬화하여 저장합니다.\
사용자당 최대 3개까지 저장할 수 있습니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Query Parameters:
- roomId: 방 ID

Body:
{
  "name": "프리셋 이름"
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
    "name": "프리셋 이름",
    "data": "[{\"type\":\"KEEP\",\"number\":1,...}]",
    "createdAt": "2026-02-22T06:00:00"
  }
}
```

---

## [실패 응답]

가능한 ErrorCode:

- ROOM_NOT_FOUND
- PRESET_LIMIT_EXCEEDED
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. 프리셋 개수 제한 검증 (최대 3개)
3. Room 조회
4. 아군 진영 성/슬롯/영웅 데이터 직렬화
5. Preset 엔티티 생성 및 저장
6. 성공 응답 반환
