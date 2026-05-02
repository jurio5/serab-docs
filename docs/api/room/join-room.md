# 방 참여 API

POST /rooms/{roomId}/join

방에 참여자로 입장하는 API입니다.

---

## [설명]

로그인된 사용자가 방에 일반 참여자(USER)로 입장합니다.
이미 같은 방의 `OFFLINE` 멤버인 경우에는
기존 `RoomMember`를 다시 `ACTIVE`로 복구합니다.
대기방에서 `owner = null`인 경우에는 입장자 또는 복귀자가 방장을 가져갈 수 있습니다.

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 참여할 방 ID

Body (비밀방인 경우):
{
  "password": "1234"
}

Body (공개방인 경우):
비어있거나 생략 가능
```
---

## [비즈니스 정책]

| 조건 | 결과 |
|------|------|
| 방이 CLOSED/LOCKED 상태 | ROOM_NOT_JOINABLE |
| 다른 `ACTIVE` / `OFFLINE` 방에 이미 참여 중 | ROOM_ALREADY_IN_ACTIVE_ROOM |
| 같은 방에 이미 `ACTIVE` 상태로 참여 중 | ROOM_ALREADY_JOINED |
| 비밀번호 불일치 | ROOM_PASSWORD_INCORRECT |
| 기존 `OFFLINE` 멤버 재입장 | 같은 `RoomMember`를 `ACTIVE`로 복구 |
| 비밀방의 기존 `OFFLINE` 멤버 재입장 | 비밀번호 재입력 없이 복귀 |

`ROOM_ALREADY_JOINED`는 같은 방에 이미 입장된 상태를 의미하며, 클라이언트에서는 재요청 상황에서 정상 흐름처럼 처리할 수 있습니다.

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
- ROOM_NOT_JOINABLE
- ROOM_ALREADY_IN_ACTIVE_ROOM
- ROOM_ALREADY_JOINED
- ROOM_PASSWORD_INCORRECT
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. Room 조회
3. 기존 RoomMember 존재 여부 확인
4. 다른 ACTIVE / OFFLINE 방 참여 여부 검증
5. 방 상태 검증 (ACTIVE만 가능)
6. 신규 참여 또는 비참여 기존 멤버인 경우 비밀번호 검증
7. `KICKED` 여부 검증
8. 기존 `OFFLINE` 멤버면 `ACTIVE`로 복구
9. 신규 참여자면 `RoomMember` 생성
10. 성공 응답 반환

---

## [실시간 이벤트]

방 참여 시 발행되는 WebSocket 이벤트는 케이스에 따라 다릅니다.

| 케이스 | 이벤트 | 채널 | 설명 |
|-------|-------|-----|-----|
| 신규 참여 | `ROOM_MEMBER_COUNT_CHANGED` | `/topic/lobby` | 로비의 멤버 수 갱신 |
| 신규 참여 | `MEMBER_JOINED` | `/topic/room/{roomId}` | 방 내부에 입장 알림 |
| `OFFLINE -> ACTIVE` 복귀 | `MEMBER_STATUS_CHANGED` | `/topic/room/{roomId}` | 오프라인 멤버 재활성화 |
| `owner = null` 상태에서 방장 지정 | `ROOM_UPDATED` | `/topic/lobby`, `/topic/room/{roomId}` | 로비 / 방 내부 방장 정보 갱신 |

