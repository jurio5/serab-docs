# Push 알림 동작 메모

채팅/공지 푸시 알림이 실제로 전달되고, 클릭 시 어디로 이동하는지 정리한 문서입니다.

---

## 구독 상태

- 브라우저 권한이 `granted`라고 해서 바로 푸시 활성 상태는 아닙니다.
- 실제 활성 상태로 보려면 아래 조건이 모두 만족되어야 합니다.
  - `Notification.permission = granted`
  - FCM 토큰 발급 성공
  - `POST /push/subscribe` 성공
- 프런트의 `알림 켜짐` 표시는 위 조건을 기준으로 맞춰집니다.

---

## 채팅 푸시 발송 조건

- room 채팅 푸시는 같은 방의 `OFFLINE` 멤버에게만 전송됩니다.
- 메시지를 보낸 발신자는 제외됩니다.
- 같은 오프라인 멤버에게는 방별 `notified` 상태가 1분 TTL로 유지됩니다.
- 따라서 짧은 시간 안에 같은 방 채팅이 연속으로 와도 매번 푸시가 가지 않을 수 있습니다.

관련 코드:

- `src/main/java/sena/core/content/room/service/ChatService.java`
- `src/main/java/sena/core/content/push/service/PushService.java`
- `src/main/java/sena/core/content/push/service/RoomPushStateService.java`
- `src/main/java/sena/core/content/room/service/RoomMemberService.java`

---

## 링크와 클릭 이동

- room 채팅 푸시는 `/rooms/{roomId}` 경로를 포함합니다.
- 실제 전송 시에는 `custom.site.frontUrl` 기준 절대 URL로 정규화합니다.
  - 예: `https://serab.dev/rooms/123`
- 서비스워커 클릭 처리 순서는 아래와 같습니다.
  - 이미 같은 room URL 창이 있으면 `focus`
  - 같은 origin 창만 있으면 해당 room URL로 `navigate` 후 `focus`
  - 열린 창이 없으면 새 창 `open`

관련 코드:

- `frontend/public/sw.js`
- `src/main/java/sena/core/content/push/service/PushService.java`

---

## Android / PWA 메모

- 설치형 PWA라도 실제 웹 푸시는 브라우저의 사이트 알림 권한을 함께 사용합니다.
- Android에서는 앱 정보 알림 허용과 Chrome 사이트 알림 허용이 모두 필요할 수 있습니다.
- `Notification.permission = denied` 상태가 되면 앱 안에서 권한 팝업을 다시 띄울 수 없고, 브라우저 설정에서 직접 허용해야 합니다.
- 카카오톡처럼 화면 상단에 잠깐 뜨는 heads-up 배너는 PWA 코드만으로 보장할 수 없어서 추가하진 않았습니다.
- 상단 상태바의 작은 아이콘은 notification `badge` 이미지를 사용하며, 운영체제가 단색/마스킹 형태로 표시할 수 있습니다.

---

## 참고

- 푸시 구독 API: `docs/api/push/push-subscription.md`
- WebSocket 채팅: `docs/websocket/websocket-chat.md`
