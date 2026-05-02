# 방 종료 API

POST /rooms/{roomId}/close

방을 종료하는 API입니다.

---

## [설명]

방장만 방을 종료할 수 있습니다.\
종료된 방은 더 이상 수정할 수 없습니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 종료할 방 ID
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
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. Room 조회
3. 방장 권한 검증
4. Room 상태를 CLOSED로 변경
5. closedAt 시간 기록
6. 성공 응답 반환
