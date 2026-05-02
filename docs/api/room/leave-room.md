# 방 퇴장 API

POST /rooms/{roomId}/leave

방에서 퇴장하는 API입니다.

---

## [설명]

로그인된 사용자가 방에서 퇴장합니다.
퇴장은 멤버십 종료이며 `OFFLINE` 전환과는 다른 흐름입니다.

방장이 퇴장할 경우 다음 규칙이 적용됩니다:
1. `ACTIVE` 후임이 있으면 → 소유권 이전 (MANAGER 우선)
2. `ACTIVE` 후임이 없고 대기방이면 → 방 유지, `owner = null`
3. `ACTIVE` 후임이 없고 활성화 방이면 → 방 자동 종료

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 퇴장할 방 ID
```
---

## [방장 퇴장 규칙]

| 조건 | 결과 |
|------|------|
| 다른 `ACTIVE` MANAGER가 있음 | MANAGER에게 방장 이전 |
| MANAGER 없고 다른 `ACTIVE` 멤버 있음 | 가장 먼저 정렬되는 `ACTIVE` 멤버에게 이전 |
| 다른 `ACTIVE` 멤버 없음 + `waitingMode = true` | `owner = null`, 방은 `ACTIVE` 유지 |
| 다른 `ACTIVE` 멤버 없음 + `waitingMode = false` | 방 자동 `CLOSED` |

`OFFLINE` 멤버는 참여 중인 멤버이지만
퇴장 시 방장 승계 대상에는 포함되지 않습니다.

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
- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. Room 조회
3. 방장 여부 확인
4. (방장인 경우) 다음 `ACTIVE` 소유자 탐색 또는 `owner = null` / 방 종료 처리
5. RoomMember 제거
6. 성공 응답 반환

---

## [실시간 이벤트]

방 퇴장 시 발행되는 WebSocket 이벤트:

| 이벤트 | 채널 | 설명 |
|-------|-----|-----|
| `ROOM_MEMBER_COUNT_CHANGED` | `/topic/lobby` | 로비의 멤버 수 갱신 |
| `ROOM_UPDATED` | `/topic/lobby`, `/topic/room/{roomId}` | 방장 변경 또는 `방장 없음` 상태 갱신 |
| `MEMBER_LEFT` | `/topic/room/{roomId}` | 방 내부에 퇴장 알림 |
| `ROOM_CLOSED` | `/topic/lobby`, `/topic/room/{roomId}` | 활성화 방에서 마지막 방장이 나가 방 종료된 경우 |

> 대기방에서 마지막 방장이 나가면 방은 유지되며,
> 로비에서는 `ownerNickname = "방장 없음"`으로 보일 수 있습니다.

