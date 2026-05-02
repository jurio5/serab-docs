# 대기방 모드 토글 API

PATCH /rooms/{roomId}/waiting-mode

방의 대기방 모드를 전환하는 API입니다.

---

## [설명]

방장만 대기방 모드를 전환할 수 있습니다.\
대기방 모드(true): 모든 인원이 나가도 방이 유지됩니다.\
활성화 모드(false): 마지막 인원이 나가면 방이 즉시 삭제됩니다.\
기본값은 대기방 모드(true)입니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 토글할 방 ID
```
---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": true
}
```

> data: 전환 후의 waitingMode 값 (true = 대기방, false = 활성화)

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
4. waitingMode 토글
5. 전환된 waitingMode 값 반환
