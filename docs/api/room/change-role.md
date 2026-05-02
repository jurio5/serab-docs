# 역할 변경 API

PUT /rooms/{roomId}/members/{memberId}/role

방 참여자의 역할을 변경하는 API입니다.

---

## [설명]

방장만 다른 참여자의 역할을 변경할 수 있습니다.\
자신의 역할은 변경할 수 없으며, OWNER 역할은 부여할 수 없습니다.

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 방 ID
- memberId: 역할을 변경할 멤버 ID

Body:
{
  "role": "MANAGER"
}
```
---

## [역할 종류]

| 역할 | 설명 | 권한 |
|------|------|------|
| OWNER | 방장 | 모든 권한 (역할 변경, 방 종료, 강퇴 등) |
| MANAGER | 부매니저 | 성 관리, 기록 수정, 강퇴 |
| USER | 일반 참여자 | 기본 권한 (조회, 기록 추가) |

---

## [비즈니스 정책]

| 조건 | 결과 |
|------|------|
| 권한 없음 (USER) | ROOM_NO_PERMISSION |
| 자신의 역할 변경 | ROOM_CANNOT_CHANGE_OWN_ROLE |
| OWNER 역할 부여 시도 | ROOM_CANNOT_ASSIGN_OWNER |

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
- ROOM_CANNOT_CHANGE_OWN_ROLE
- ROOM_CANNOT_ASSIGN_OWNER
- ROOM_ROLE_INVALID
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. ChangeRole VO 생성 → 역할 검증
3. 요청자 권한 확인 (OWNER만 가능)
4. 자기 자신 변경 불가 검증
5. OWNER 역할 부여 불가 검증
6. 대상 멤버 역할 변경
7. 성공 응답 반환

---

## [실시간 이벤트]

역할 변경 시 발행되는 WebSocket 이벤트:

| 이벤트 | 채널 | 설명 |
|-------|-----|-----|
| `MEMBER_ROLE_CHANGED` | `/topic/room/{roomId}` | 역할 변경 알림 |

이벤트 데이터:
```json
{
  "memberId": 123,
  "newRole": "MANAGER",
  "nickname": "테스터1"
}
```

