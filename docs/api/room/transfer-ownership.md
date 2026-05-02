# 방장 양도 API

POST /rooms/{roomId}/transfer/{memberId}

방장 권한을 다른 멤버에게 양도하는 API입니다.

---

## [설명]

방장만 다른 멤버에게 방장 권한을 양도할 수 있습니다.\
양도 후 기존 방장은 매니저로 변경됩니다.\
이 작업은 되돌릴 수 없습니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
- memberId: 양도 대상 멤버 ID
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
- ROOM_MEMBER_NOT_FOUND
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. Room 조회
3. 방장 검증 (요청자 == 방장)
4. 대상 멤버 존재 확인
5. Room.owner를 대상 멤버로 변경
6. 대상 멤버 role → OWNER
7. 기존 방장 role → MANAGER
8. WebSocket으로 양쪽 역할 변경 이벤트 발행
9. 성공 응답 반환
