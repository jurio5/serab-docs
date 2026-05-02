# 블랙리스트 조회 API

GET /rooms/{roomId}/blacklist

방에서 강퇴된 멤버 목록을 조회하는 API입니다.

---

## [설명]

해당 방에서 강퇴(KICKED) 상태인 멤버 목록을 반환합니다.\
블랙리스트에 등록된 멤버는 방에 재입장이 불가합니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
```
---

## [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": [
    {
      "id": 10,
      "memberId": 3,
      "nickname": "테스터1",
      "role": "USER",
      "isCurrentUser": false
    }
  ]
}
```

> `isCurrentUser`는 블랙리스트 조회 시 항상 `false`로 반환됩니다.

---

## [실패 응답]

가능한 ErrorCode:

- ROOM_NOT_FOUND
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. Room 존재 여부 확인
3. 상태가 KICKED인 RoomMember 목록 조회
4. RoomMemberResponse 변환 후 반환
