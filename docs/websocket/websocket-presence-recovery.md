# WebSocket 접속 상태 복구

WebSocket 접속 상태 복구 흐름을 정리한 문서입니다.  
이 문서는 `접속 상태(presence)` 와 `RoomMember 도메인 상태`를 분리해서 설명합니다.

---

## 문서 개요

- Redis 기반 접속 상태 저장 방식
- `CONNECT`, `PING`, `DISCONNECT` 흐름
- OFFLINE 전이를 위한 유예 정책
- `global.websocket` 과 `content.room` 경계를 이벤트로 분리한 이유

---

## 핵심 개념

### 접속 상태

여기서 말하는 `presence` 는 `접속 상태`를 뜻합니다.

- Redis 세션 키가 살아 있으면 접속 중으로 봅니다.
- 세션 키가 사라졌다고 바로 OFFLINE 처리하지 않습니다.
- 유예 시간이 지난 뒤에도 복구되지 않았을 때만 `RoomMember.status = OFFLINE` 으로 전이합니다.

### 도메인 상태

방 참여자의 상태는 `RoomMemberStatus` 로 관리합니다.

- `ACTIVE`
- `OFFLINE`
- `KICKED`

즉 Redis 접속 상태와 `RoomMember` 상태는 같은 개념이 아닙니다.

---

## 현재 값

| 항목 | 값 |
| --- | --- |
| WebSocket reconnect delay | 5초 |
| presence ping 주기 | 10초 |
| Redis 세션 TTL | 45초 |
| disconnect grace | 10초 |
| stale active 보정 주기 | 30초 |

---

## 처리 흐름

### 1. CONNECT

`WebSocketEventListener.handleWebSocketConnectListener()`

- 웹소켓 연결이 열리면 `memberId`, `sessionId` 를 확인합니다.
- `WebSocketSessionRegistry.addConnectedSession()` 으로 Redis 세션 키를 등록합니다.
- 이후 `MemberPresenceRecoveredPayload` 를 발행합니다.

이 단계의 목적은 `이 사용자의 접속 상태가 살아 있다`는 신호를 만드는 것입니다.

### 2. PING

`WebSocketPresenceController.ping()`

- 프론트는 `/app/presence/ping` 으로 10초마다 ping을 보냅니다.
- 서버는 `refreshConnectedSession()` 으로 TTL을 갱신합니다.
- Redis에 세션 키가 이미 있으면 단순 갱신만 수행합니다.
- Redis에 세션 키가 없었다면, 접속 상태가 끊겼다가 복구된 것으로 보고 `MemberPresenceRecoveredPayload` 를 발행합니다.

즉 ping은 keepalive이면서 동시에 접속 상태 복구 감지 역할도 합니다.

### 3. DISCONNECT

`WebSocketEventListener.handleWebSocketDisconnectListener()`

- 세션을 제거합니다.
- 마지막 세션까지 사라졌더라도 바로 OFFLINE 처리하지 않습니다.
- `scheduleOffline()` 으로 pending offline 상태만 등록합니다.

### 4. pending offline 확인

`WebSocketRoomPresenceScheduler`

- 3초마다 pending offline 대상을 확인합니다.
- 유예 시간이 지났고 그 사이 접속 상태가 복구되지 않았다면 `markRoomsOffline()` 을 호출합니다.

### 5. stale ACTIVE 보정

`WebSocketActiveMemberReconciliationScheduler`

- 30초마다 DB의 `ACTIVE` 멤버와 Redis 접속 상태를 비교합니다.
- 세션이 없는 멤버를 발견해도 즉시 OFFLINE 처리하지 않습니다.
- 여기서도 먼저 `scheduleOffline()` 으로 대기 상태만 등록합니다.

이 경로는 disconnect 이벤트를 놓쳤을 때의 보정 경로입니다.

---

## 왜 바로 OFFLINE 처리하지 않는가

모바일 브라우저, 백그라운드 탭, 일시적인 네트워크 흔들림에서는 heartbeat와 ping이 잠깐 밀릴 수 있습니다.

이 상황에서 즉시 OFFLINE 처리하면 다음 문제가 생깁니다.

- 실제로는 다시 붙을 사용자를 OFFLINE 으로 잘못 바꿀 수 있습니다.
- 방장 위임 같은 도메인 로직이 불안하게 실행될 수 있습니다.
- 사용자 화면에는 `방에 있는데 오프라인` 으로 보일 수 있습니다.

그래서 현재 구조는 `끊김 감지 -> pending -> 재확인 -> OFFLINE 확정` 순서를 사용합니다.

---

## 왜 CONNECT 와 PING 모두 복구 신호를 발행하는가

복구 타이밍을 빠르게 잡기 위해서입니다.

- CONNECT 는 새 웹소켓 연결이 열리면 즉시 복구를 시도합니다.
- PING 은 Redis 세션 키가 사라져 있었던 경우를 다시 감지해 복구를 시도합니다.

둘 중 하나만 있으면 복구 타이밍이 늦어지거나, 특정 경우에 복구를 놓칠 수 있습니다.

---

## 경계 분리

초기 구현은 `global.websocket` 에서 `RoomMemberService` 를 직접 호출하는 구조였습니다.

이 구조는 다음 문제가 있었습니다.

- `global` 레이어가 `room` 도메인에 직접 의존합니다. (SRP)
- WebSocket 연결 처리와 Room 도메인 복구 책임이 한 클래스에 얽힙니다.
- 복구 정책 변경이 `global.websocket` 수정으로 이어집니다.

현재는 이를 이벤트로 분리했습니다.

### 발행자

- `WebSocketEventListener`
- `WebSocketPresenceController`

두 곳은 공통으로 `MemberPresenceRecoveredPayload` 만 발행합니다.

### 수신자

- `RoomPresenceRecoveryListener`

이 리스너가 payload를 받아 `RoomMemberService.recoverOfflineMemberships()` 를 호출합니다.

즉 global은 `접속 상태가 복구되었다`는 사실만 알리고, room은 그 신호를 받아 도메인 복구를 수행합니다.

---

## 관련 파일

### WebSocket

- `src/main/java/sena/core/global/websocket/WebSocketEventListener.java`
- `src/main/java/sena/core/global/websocket/WebSocketSessionRegistry.java`
- `src/main/java/sena/core/global/websocket/controller/WebSocketPresenceController.java`
- `src/main/java/sena/core/global/websocket/dto/MemberPresenceRecoveredPayload.java`

### Room

- `src/main/java/sena/core/content/room/listener/RoomPresenceRecoveryListener.java`
- `src/main/java/sena/core/content/room/service/RoomMemberService.java`
- `src/main/java/sena/core/content/room/scheduler/WebSocketRoomPresenceScheduler.java`
- `src/main/java/sena/core/content/room/scheduler/WebSocketActiveMemberReconciliationScheduler.java`

### Frontend

- `frontend/src/lib/stompClient.ts`

---

## 관련 테스트

- `WebSocketEventListenerTest`
- `WebSocketActiveMemberReconciliationSchedulerTest`
- `RoomPresenceRecoveryListenerTest`
- `RoomMemberServiceTest`

이 문서를 읽을 때 가장 중요한 기준은 다음과 같습니다.

`접속 상태 복구와 도메인 상태 복구는 같은 일이 아니다`

전자는 WebSocket/Redis의 책임이고, 후자는 Room 도메인의 책임입니다.
