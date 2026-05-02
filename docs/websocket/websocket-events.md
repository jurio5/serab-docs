# WebSocket 이벤트 문서

실시간 동기화를 위한 WebSocket(STOMP) 이벤트 문서입니다.

---

## 연결 정보

| 항목 | 값 |
|-----|-----|
| 엔드포인트 | `/ws` |
| 프로토콜 | STOMP over WebSocket |
| SockJS 지원 | ✅ |

---

## 구독 채널

### 로비 (방 목록)

| Destination | 설명 |
|------------|-----|
| `/topic/lobby` | 방 목록 변경 알림 |

### 방 내부

| Destination | 설명 |
|------------|-----|
| `/topic/room/{roomId}` | 특정 방의 모든 이벤트 |

---

## 발행 채널 (Client → Server)

| Destination | 설명 |
|------------|-----|
| `/app/room/{roomId}/chat` | 채팅 메시지 전송 |

---

## 이벤트 타입

### 메시지 형식

```json
{
  "type": "EVENT_TYPE",
  "data": { ... },
  "timestamp": 1705759200000
}
```

### 이벤트 목록

| 타입 | 설명 | data |
|-----|-----|-----|
| `ROOM_CREATED` | 로비에 새 방 추가 | `RoomResponse` |
| `ROOM_CLOSED` | 방 종료 | `roomId (Long)` |
| `ROOM_UPDATED` | 방 설정 / 방장 정보 갱신 | `RoomResponse` |
| `ROOM_MEMBER_COUNT_CHANGED` | 로비 참여자 수 갱신 | `{ roomId, memberCount }` |
| `CASTLE_UPDATED` | 성 정보 변경 | `CastleResponse` |
| `DEFENDER_ADDED` | 방어팀 추가 | `DefenderSlotResponse` |
| `DEFENDER_UPDATED` | 방어팀 수정 | `DefenderSlotResponse` |
| `MEMBER_JOINED` | 멤버 입장 / observer → participant 전환 | `RoomMemberResponse` |
| `MEMBER_LEFT` | 멤버 퇴장 | `{ memberId, nickname }` |
| `MEMBER_ROLE_CHANGED` | 역할 변경 | `{ memberId, newRole, nickname }` |
| `MEMBER_STATUS_CHANGED` | `ACTIVE / OFFLINE` 상태 변경 | `{ memberId, status, nickname }` |
| `MEMBER_KICKED` | 멤버 강퇴 | `{ memberId, nickname }` |
| `BATTLE_RECORDED` | 격파 기록 추가 | `BattleRecordResponse` |
| `BATTLE_UPDATED` | 격파 기록 수정 | `BattleRecordResponse` |
| `BATTLE_DELETED` | 격파 기록 삭제 | `battleRecordId (Long)` |
| `CHAT_MESSAGE` | 채팅 메시지 | `ChatMessage` |
| `SCORE_UPDATED` | 아군 / 적군 점수 갱신 | `{ allyScore, enemyScore }` |
| `PRESET_LOADED` | 프리셋 불러오기 완료 | `null` |

---

## 사용 예시 (Frontend)

### 연결

```javascript
const socket = new SockJS('/ws');
const stompClient = Stomp.over(socket);

stompClient.connect({}, () => {
  console.log('Connected');
});
```

### 로비 구독

```javascript
stompClient.subscribe('/topic/lobby', (message) => {
  const event = JSON.parse(message.body);
  if (event.type === 'ROOM_UPDATED') {
    updateRoomCard(event.data);
  }
  if (event.type === 'ROOM_MEMBER_COUNT_CHANGED') {
    updateRoomMemberCount(event.data.roomId, event.data.memberCount);
  }
});
```

### 방 내부 구독

```javascript
stompClient.subscribe(`/topic/room/${roomId}`, (message) => {
  const event = JSON.parse(message.body);
  
  switch (event.type) {
    case 'DEFENDER_ADDED':
      addDefenderToUI(event.data);
      break;
    case 'MEMBER_JOINED':
      addMember(event.data);
      break;
    case 'MEMBER_STATUS_CHANGED':
      updateMemberStatus(event.data.memberId, event.data.status);
      break;
    // ...
  }
});
```
