# 채팅 히스토리 조회 API

GET /rooms/{roomId}/chats

방의 채팅 히스토리를 조회하는 API입니다.

---

## [설명]

방 입장 시 이전 채팅 기록을 로드합니다.
Redis에 데이터가 있으면 Redis에서, 없으면 MySQL에서 조회합니다 (Write-Behind 패턴).

---

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...

Path Parameters:
- roomId: 조회할 방 ID
```
---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": [
    {
      "type": null,
      "senderId": 1,
      "senderNickname": "닉네임",
      "content": "안녕하세요",
      "role": "USER",
      "timestamp": 1711440000000
    },
    {
      "type": "system",
      "senderId": 0,
      "senderNickname": "",
      "content": "닉네임님이 입장했습니다",
      "role": null,
      "timestamp": 1711440001000
    }
  ]
}
```

### 필드 설명

| 필드 | 타입 | 설명 |
|---|---|---|
| type | String? | `null`: 일반 메시지, `"system"`: 시스템 메시지 (입퇴장 등) |
| senderId | Long | 발신자 ID (시스템 메시지는 0) |
| senderNickname | String | 발신자 닉네임 (시스템 메시지는 빈 문자열) |
| content | String | 메시지 내용 (최대 200자) |
| role | String? | 발신자 역할 (`OWNER`, `MANAGER`, `USER`), 시스템 메시지는 null |
| timestamp | long | 메시지 전송 시각 (Unix Epoch 밀리초) |

---

## [실패 응답]

가능한 ErrorCode:

- UNAUTHORIZED

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. Redis 키(`chat:room:{roomId}`) 존재 여부 확인
3. Redis에 있으면 → Redis List 전체 조회 (ACTIVE/CLOSED 상태)
4. Redis에 없으면 → MySQL `chat_message` 테이블 fallback (LOCKED 이후)
5. ChatMessageResponse 변환 후 응답 반환

---

## [관련 아키텍처]

### Write-Behind 패턴

```
채팅 전송 → Redis RPUSH (최대 500개)
                ↓
         WebSocket broadcast
                ↓
방 LOCKED 시 → JDBC Batch INSERT → MySQL
                ↓
           Redis DEL
```

- 실시간 저장: Redis List (`O(1)`)
- 배치 영속화: RoomScheduler가 화/목/일 03:00에 실행
- 방당 메시지 상한: 500개 (LTRIM으로 유지)
