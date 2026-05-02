# 차단 해제 API

DELETE /rooms/{roomId}/blacklist/{memberId}

강퇴된 멤버의 차단을 해제하는 API입니다.

---

## [설명]

블랙리스트에서 멤버를 제거합니다.\
차단이 해제된 멤버는 다시 방에 입장할 수 있게 됩니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
- memberId: 차단 해제할 멤버 ID
```
---

## [비즈니스 정책]

| 조건 | 결과 |
|------|------|
| 권한 없음 (USER) | ROOM_NO_PERMISSION |
| 대상이 KICKED 상태가 아님 | ROOM_MEMBER_NOT_FOUND |
| 이미 차단 해제된 멤버 | ROOM_MEMBER_NOT_FOUND |

> OWNER와 MANAGER(canEditBattle 권한)만 차단 해제가 가능합니다.

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
- ROOM_MEMBER_NOT_FOUND
- ROOM_NO_PERMISSION
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. 요청자 권한 확인 (canEditBattle)
3. 대상 멤버가 KICKED 상태인지 검증
4. Room의 members 컬렉션에서 제거 (orphanRemoval로 DB 삭제)
5. 성공 응답 반환
