# WebSocket 문서 목차

실시간 동기화를 위한 WebSocket(STOMP) 관련 문서입니다.

---

## 문서 목록

- [WebSocket 이벤트](./websocket-events.md) - 이벤트 타입 및 구독 채널
- [채팅](./websocket-chat.md) - 실시간 채팅 (양방향 STOMP)
- [Disconnect 처리](./websocket-disconnect-handling.md) - 오프라인 전환과 재연결 유예
- [티켓 기반 인증](./websocket-ticket-auth.md) - 일회용 티켓 인증 방식

---

## 개요

| 항목 | 설명 |
|-----|-----|
| 프로토콜 | STOMP over WebSocket |
| 엔드포인트 | `/ws` |
| 메시지 브로커 | SimpleBroker (내장) |
| 확장 계획 | RabbitMQ (추후) |

---

## 아키텍처

```
Client                              Server
  │                                   │
  ├── /ws (SockJS) ─────────────────► │ WebSocket 연결
  │                                   │
  ├── SUBSCRIBE /topic/lobby ───────► │ 로비 구독
  ├── SUBSCRIBE /topic/room/123 ────► │ 방 구독
  │                                   │
  │ ◄── ROOM_LIST_CHANGED ─────────── │ 방 목록 변경 알림
  │ ◄── DEFENDER_ADDED ────────────── │ 방어팀 추가 알림
  │ ◄── CHAT_MESSAGE ──────────────── │ 채팅 메시지 수신
  │                                   │
  ├── PUBLISH /app/room/123/chat ───► │ 채팅 메시지 발신 (양방향)
  │                                   │
```
