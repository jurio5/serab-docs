# WebSocket Disconnect 처리

방 입장 이후의 WebSocket disconnect 처리와 `ACTIVE` / `OFFLINE` 상태 전환 기준입니다.
채팅 FCM 푸시 대상도 이 기준을 사용합니다.

---

## 개요

| 항목 | 설명 |
|-----|------|
| 재연결 유예 시간 | 3초 |
| `ACTIVE` 전환 | 방 참여 성공 또는 `OFFLINE -> ACTIVE` 복귀 시 |
| `OFFLINE` 전환 | disconnect 후 재연결이 없을 때 |
| 보정 스케줄러 | 30초마다 stale `ACTIVE`를 `OFFLINE`으로 정리 |
| leave / kick | 멤버십 종료 또는 `KICKED` 처리 |
| 채팅 푸시 대상 | 같은 방의 `OFFLINE` 멤버 (발신자 제외) |
| 참여자 패널 표시 | `OFFLINE` 멤버는 역할 대신 `오프라인`으로 표시 |

---

## 상태 모델

| 상태 | 의미 |
|-----|------|
| `ACTIVE` | 현재 방에 참여 중인 상태 |
| `OFFLINE` | 방 멤버십은 유지되지만 WebSocket 연결이 끊긴 상태 |
| `KICKED` | 강퇴된 상태 |

`ACTIVE`와 `OFFLINE`은 모두 방에 속한 멤버로 취급합니다.
즉, disconnect는 퇴장이 아니라 상태 전환입니다.

프론트는 `/rooms/{roomId}/members` 응답의 `status`와
`MEMBER_STATUS_CHANGED` 이벤트를 함께 사용해 참여자 목록을 동기화합니다.

---

## 기존 방식과 변경점

기존에는 disconnect 이후 방에서 바로 제거하는 흐름이 있었습니다.
현재는 아래 기준으로 동작합니다.

- 방 입장 시 `ACTIVE`
- leave / kick 시 멤버십 종료
- disconnect 시 즉시 퇴장하지 않고 3초 유예 후 `OFFLINE`

즉, disconnect는 방 제거가 아니라 `ACTIVE -> OFFLINE` 전환입니다.

---

## 처리 흐름

### 1. 방 입장

- 신규 참여자면 `RoomMember`를 생성합니다.
- 기존 `OFFLINE` 멤버면 같은 `RoomMember`를 비밀번호 재검증 없이 다시 `ACTIVE`로 전환합니다.
- 푸시 딥링크 등으로 방 상세에 먼저 진입한 경우,
  현재 사용자의 `status = OFFLINE`이면 클라이언트가 `POST /rooms/{roomId}/join`을 한 번 더 호출해 재활성화합니다.

### 2. WebSocket connect

- `WebSocketEventListener`가 `memberId`, `sessionId`를 읽습니다.
- `WebSocketSessionRegistry`에 연결 세션을 등록합니다.
- 같은 멤버의 pending offline 예약이 있으면 제거합니다.

### 3. WebSocket disconnect

- 세션을 제거합니다.
- 더 이상 남은 연결 세션이 없으면 `now + 3초`를 통해 offline 후보로 저장합니다.

### 4. 스케줄러 확인

- `WebSocketRoomPresenceScheduler`가 1초마다 pending offline 대상을 확인합니다.
- 유예 시간이 지났고 재연결이 없으면 `RoomMemberService.markRoomsOffline(memberId)`를 호출합니다.

즉, 페이지 이동이나 일시적인 재연결 상황에서 바로 `OFFLINE`으로 바꾸지 않기 위해 3초 유예를 둡니다.

### 5. 주기 보정 스케줄러

- `WebSocketActiveMemberReconciliationScheduler`가 30초마다 실행됩니다.
- DB의 `ACTIVE room_member` 집합과 실제 live WebSocket `memberId` 집합을 비교합니다.
- DB에는 `ACTIVE`지만 live 세션이 없는 멤버만 `RoomMemberService.markRoomsOffline(memberId)`로 보정합니다.

이 보정은 아래 상황에서 disconnect 이벤트를 놓쳤더라도 stale `ACTIVE`를 정리하기 위한 백업 경로입니다.

- 브라우저 / 앱 강제 종료
- 네트워크 단절
- 서버 재시작 이후 남아 있는 `ACTIVE`

---

## 채팅 푸시 기준

채팅 메시지는 먼저 WebSocket으로 브로드캐스트하고, 이후 FCM 푸시를 보냅니다.

방 채팅 푸시는 아래 조건을 모두 만족하는 대상에게만 보냅니다.

- 같은 방 멤버
- `RoomMemberStatus.OFFLINE`
- 발신자가 아님

즉 아래 대상은 room 푸시를 받지 않습니다.

- 현재 방에 있는 `ACTIVE` 멤버
- leave 한 멤버
- 강퇴된 멤버

채팅 푸시는 같은 오프라인 사이클에서 무제한으로 보내지지 않도록 제어합니다.

- `offlineMembers` : 현재 방에서 `OFFLINE` 상태인 멤버 집합
- `notifiedMembers` : 현재 오프라인 사이클에서 이미 채팅 푸시를 보낸 멤버 집합
- 실제 발송 대상 : `offlineMembers - notifiedMembers - sender`

첫 채팅에서는 Redis 기준으로 대상을 계산한 뒤
`PushSubscriptionRepository.findTokensByMemberIds(...)`로 토큰을 조회합니다.
발송이 성공하면 해당 멤버를 `notifiedMembers`에 기록하므로,
같은 방에서 바로 이어지는 다음 채팅부터는 추가 조회 없이 종료됩니다.

`notifiedMembers`는 방별 10분 TTL로 만료됩니다.
즉, 사용자가 아직 방에 다시 들어오지 않았더라도
10분이 지난 뒤 다음 채팅이 오면 상기용 푸시를 다시 받을 수 있습니다.

채팅 푸시에는 `/rooms/{roomId}` 딥링크가 함께 포함됩니다.
알림 클릭 시 서비스워커는 아래 순서로 처리합니다.

- 이미 같은 room URL 창이 있으면 해당 창을 focus
- 같은 origin 창만 있으면 target room URL로 navigate 후 focus
- 해당 창이 없으면 새 창으로 room URL open

아래 이벤트가 발생하면 room 푸시 상태를 초기화합니다.

- 방 재입장 (`OFFLINE -> ACTIVE`)
- leave
- kick

공지사항 푸시는 room 채팅과 별개로 동작하며,
기존처럼 제목과 본문 preview를 그대로 발송합니다.

---

## 프론트 동기화

- `/rooms/{roomId}/members` 응답은 `status`를 포함합니다.
- `MEMBER_STATUS_CHANGED` 이벤트로 `ACTIVE / OFFLINE` 전환을 즉시 반영합니다.
- 참여자 패널은 `OFFLINE` 멤버를 역할 대신 `오프라인`으로 표시합니다.
- 로비 카드는 `ROOM_UPDATED` 이벤트로 `ownerNickname`과 방 설정을 즉시 갱신합니다.

---

## 방장 처리

방장 처리 정책은 `leave`와 `offline`을 분리해서 봅니다.

### leave

- 방장이 퇴장할 때 `ACTIVE` 후임이 있으면 방장 승계 (`MANAGER` 우선)
- 대기방이고 `ACTIVE` 후임이 없으면 `owner = null`
- 활성화 방이고 `ACTIVE` 후임이 없으면 방 종료

### offline

- 방장이 `OFFLINE` 되면 `ACTIVE` 후임이 있을 때만 방장 승계
- 승계되면 새 방장은 `OWNER`, 기존 방장은 `USER`
- `ACTIVE` 후임이 없고 방 안에 `ACTIVE` 유저가 한 명도 없으면 `owner = null`로 전환
- 즉, 참여자 수는 유지되더라도 `ACTIVE` 유저가 없으면 로비와 방 내부 모두 `방장 없음` 상태가 됩니다.

`OFFLINE` 멤버는 참여 중인 멤버이지만 자동 방장 승계 대상은 아닙니다.
로비 카드의 `ownerNickname`도 `ROOM_UPDATED` 이벤트로 즉시 동기화됩니다.

---

## 관련 클래스

### Backend
- `WebSocketEventListener.java` - connect / disconnect 이벤트 처리
- `WebSocketSessionRegistry.java` - 연결 세션 / pending offline 상태 관리
- `WebSocketRoomPresenceScheduler.java` - 3초 유예 후 offline 확정
- `WebSocketActiveMemberReconciliationScheduler.java` - 30초마다 stale `ACTIVE` 보정
- `RoomMemberService.java` - `markRoomsOffline()` 처리
- `RoomPushStateService.java` - room 채팅 푸시 1회 발송 상태 관리
- `PushService.java` - room 채팅 푸시 발송
- `PushSubscriptionRepository.java` - room 푸시 대상 멤버의 FCM 토큰 조회
