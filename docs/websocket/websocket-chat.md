# WebSocket 채팅 (Chat)

방 내 실시간 채팅 기능입니다. STOMP 양방향 통신을 사용합니다.

---

## 개요

| 항목 | 설명 |
|-----|------|
| 프로토콜 | STOMP `@MessageMapping` (양방향) |
| 저장 방식 | 인메모리 (비영속, 새로고침 시 초기화) |
| 오프라인 푸시 | 같은 방의 `OFFLINE` 멤버에게 FCM 발송 |
| 딥링크 | 채팅 푸시는 `/rooms/{roomId}` 링크 포함 |
| 확장 계획 | RabbitMQ 전환 시 메시지 큐잉 가능 |

---

## 메시지 흐름

```
Client                              Server
  │                                   │
  ├── PUBLISH /app/room/{id}/chat ──► │ ChatController.handleChat()
  │   body: "안녕!"                    │   → ChatService.sendMessage()
  │                                   │   → MemberPrincipal에서 닉네임 추출 (DB 조회 없음)
  │                                   │   → messageBroker.publish()
  │                                   │   → pushService.sendToRoom()
  │ ◄── CHAT_MESSAGE ──────────────── │ /topic/room/{id} 로 브로드캐스트
  │                                   │   → OFFLINE 멤버에게 FCM 푸시 (+ room 딥링크)
  │                                   │
```

---

## 발행 (Client → Server)

| 항목 | 값 |
|-----|------|
| Destination | `/app/room/{roomId}/chat` |
| Body | 메시지 텍스트 (String, 최대 200자) |
| 인증 | `MemberPrincipal` (STOMP 연결 시 티켓 인증) |

### 프론트엔드 예시

```typescript
import { sendChatMessage } from '@/lib/stompClient';

sendChatMessage(roomId, "안녕하세요!");
// → client.publish({ destination: `/app/room/${roomId}/chat`, body: content })
```

---

## 수신 (Server → Client)

`/topic/room/{roomId}` 구독 중인 모든 클라이언트에게 브로드캐스트됩니다.

### 이벤트 형식

```json
{
  "type": "CHAT_MESSAGE",
  "data": {
    "senderId": 1,
    "senderNickname": "테스터",
    "content": "안녕하세요!",
    "timestamp": 1739406000000
  },
  "timestamp": 1739406000000
}
```

### 프론트엔드 처리

```typescript
// useRoomSync.ts
case 'CHAT_MESSAGE':
  options.onChatMessage?.(event.data);
  break;
```

---

## 채팅 푸시

WebSocket 브로드캐스트 후 같은 방의 `OFFLINE` 멤버에게만 FCM 푸시를 보냅니다.

- 발신자는 제외됩니다.
- 같은 오프라인 사이클에서도 방별 `notified` 상태는 10분 TTL로 만료됩니다.
- 따라서 첫 푸시 후 바로 연속 발송되지는 않지만, 10분이 지난 뒤 다음 채팅부터는 상기용 푸시가 다시 갈 수 있습니다.
- 푸시 payload에는 `/rooms/{roomId}` 딥링크가 포함됩니다.

알림 클릭 시 서비스워커는 아래 순서로 이동을 처리합니다.

- 이미 같은 room URL 창이 있으면 focus
- 같은 origin 창만 있으면 해당 room URL로 navigate 후 focus
- 열려 있는 창이 없으면 새 창 open

푸시를 통해 방에 들어온 사용자가 아직 `OFFLINE` 멤버라면,
방 페이지는 첫 `detail` 응답에서 자신의 `status`를 확인한 뒤
`POST /rooms/{roomId}/join`을 다시 호출해 `ACTIVE`로 복구합니다.

### Android PWA 권한 메모

- 설치형 PWA라도 웹 푸시는 브라우저의 사이트 알림 권한을 함께 사용합니다.
- Android에서는 앱 정보의 알림 허용과 Chrome 사이트 알림 허용이 모두 필요할 수 있습니다.
- `Notification.permission = denied` 상태가 되면 앱 안에서 시스템 권한 팝업을 다시 열 수 없고,
  브라우저의 사이트 설정에서 직접 허용해야 합니다.

---

## 관련 파일

### Backend
- `ChatController.java` — `@MessageMapping` 엔드포인트 (principal에서 nickname 전달)
- `ChatService.java` — DTO 조립, 브로드캐스트
- `PushService.java` — `OFFLINE` 멤버 대상 room 채팅 푸시
- `ChatMessage.java` — DTO (`senderId`, `senderNickname`, `content`, `timestamp`)
- `RoomEventType.java` — `CHAT_MESSAGE` 타입

### Frontend
- `stompClient.ts` — `sendChatMessage()` 함수
- `useRoomSync.ts` — `onChatMessage` 콜백 핸들링
- `ChatPanel.tsx` — 채팅 UI 컴포넌트
- `public/sw.js` — 푸시 클릭 시 room 딥링크 이동
