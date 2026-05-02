# 멤버 강퇴 API

POST /rooms/{roomId}/kick/{memberId}

방 참여자를 강퇴하는 API입니다.

---

## [설명]

방장(OWNER) 또는 매니저(MANAGER)가 다른 참여자를 강퇴할 수 있습니다.\
강퇴된 멤버는 해당 방에 다시 입장할 수 없으며, 블랙리스트에 등록됩니다.

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
- memberId: 강퇴할 멤버 ID
```
---

## [강퇴 권한]

| 요청자 역할 | 대상 역할 | 결과 |
|------------|----------|------|
| OWNER | MANAGER, USER | ✅ 강퇴 가능 |
| MANAGER | USER | ✅ 강퇴 가능 |
| MANAGER | MANAGER, OWNER | ❌ ROOM_CANNOT_KICK_TARGET |
| USER | 모두 | ❌ ROOM_NO_PERMISSION |

---

## [비즈니스 정책]

| 조건 | 결과 |
|------|------|
| 자기 자신 강퇴 | ROOM_CANNOT_KICK_SELF |
| 권한 없음 (USER) | ROOM_NO_PERMISSION |
| 대상이 상위 역할 (매니저가 매니저/방장 강퇴) | ROOM_CANNOT_KICK_TARGET |
| 대상이 방 멤버가 아님 | ROOM_MEMBER_NOT_FOUND |

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
- ROOM_CANNOT_KICK_SELF
- ROOM_CANNOT_KICK_TARGET
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. 요청자 권한 확인 (OWNER 또는 MANAGER)
3. 자기 자신 강퇴 불가 검증
4. 대상 멤버 역할에 따른 강퇴 가능 여부 검증
5. 대상 멤버 상태를 KICKED로 변경
6. 성공 응답 반환

---

## [실시간 이벤트]

강퇴 시 발행되는 WebSocket 이벤트:

| 이벤트 | 채널 | 설명 |
|-------|-----|-----|
| `ROOM_LIST_CHANGED` | `/topic/lobby` | 로비의 멤버 수 갱신 |
| `MEMBER_KICKED` | `/topic/room/{roomId}` | 방 내부에 강퇴 알림 |

이벤트 데이터:
```json
{
  "memberId": 123
}
```

> 프론트엔드에서 `MEMBER_KICKED` 이벤트를 수신하면, 강퇴 대상 본인에게는 알림 모달 후 홈으로 리다이렉트합니다.
