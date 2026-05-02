# 프리셋 방에 적용 API

POST /rooms/{roomId}/presets/{presetId}/load

저장된 프리셋을 방에 적용하는 API입니다.

---

## [설명]

방장만 프리셋을 방에 적용할 수 있습니다.\
적용 시 아군 진영의 성/슬롯/영웅 배치가 프리셋 데이터로 덮어씌워집니다.\
적용 후 WebSocket으로 `PRESET_LOADED` 이벤트가 발행됩니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
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

- ROOM_NOT_FOUND
- ROOM_NOT_OWNER
- PRESET_NOT_FOUND
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. Room 조회
3. 방장 검증
4. 프리셋 조회
5. 프리셋 데이터를 아군 성에 적용 (기존 슬롯 제거 → 새 데이터 적용)
6. `PRESET_LOADED` WebSocket 이벤트 발행
7. 성공 응답 반환

---

## [실시간 이벤트]

프리셋 적용 시 발행되는 WebSocket 이벤트:

| 이벤트 | 채널 | 설명 |
|-------|-----|------|
| `PRESET_LOADED` | `/topic/room/{roomId}` | 프리셋 적용 알림 |
