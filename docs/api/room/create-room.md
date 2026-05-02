# 방 생성 API

POST /rooms

새로운 길드전 전략실을 생성하는 API입니다.

---

## [설명]

로그인된 사용자가 길드전 전략실(방)을 생성합니다.

방 생성 시 다음 작업이 수행됩니다:

- 방 생성 (제목, 현재 날짜, 생성자, 비밀번호)
- 생성자를 방장(OWNER)으로 등록
- 18개 성 자동 생성
  - 외성: 아군 5개 + 상대 5개 = 10개
  - 내성: 아군 3개 + 상대 3개 = 6개
  - 본성: 아군 1개 + 상대 1개 = 2개

생성 요청자는 현재 ACTIVE 상태의 방에 ACTIVE 멤버로 이미 참여 중이면 새 방을 생성할 수 없습니다.

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Body:
{
  "title": "1월 6일 길드전",
  "password": "1234"  // 선택, null -> 공개방
}
```
---

## [제목 유효성 규칙]

- 필수: 빈 값 불가
- 길이: 100자 이하
- 공백: 앞/뒤 공백 자동 제거

## [비밀번호 유효성 규칙]

- 선택: null 가능 (공개방)
- 길이: 4자 이상 100자 이하

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "id": 1,
    "title": "1월 6일 길드전",
    "guildWarDate": "2026-01-06",
    "ownerNickname": "닉네임",
    "status": "ACTIVE",
    "memberCount": 1,
    "hasPassword": true,
    "createdAt": "2026-01-06T23:55:00"
  }
}
```

---

## [실패 응답]

가능한 ErrorCode:

- ROOM_TITLE_INVALID
- ROOM_TITLE_TOO_LONG
- ROOM_PASSWORD_TOO_SHORT
- ROOM_PASSWORD_TOO_LONG
- ROOM_ALREADY_IN_ACTIVE_ROOM
- UNAUTHORIZED

예시:

```json
{
  "success": false,
  "errorCode": "ROOM_TITLE_INVALID",
  "message": "방 제목은 필수입니다."
}
```
---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. 현재 사용자의 활성 방 참여 여부 검증
3. CreateRoom VO 생성 → 제목, 비밀번호 검증
4. Room 엔티티 생성 (제목, 날짜, 방장, 비밀번호)
5. 18개 Castle 자동 생성
6. 생성자를 OWNER로 RoomMember 추가
7. 성공 응답 반환

---

## [실시간 이벤트]

방 생성 시 발행되는 WebSocket 이벤트:

| 이벤트 | 채널 | 설명 |
|-------|-----|-----|
| `ROOM_LIST_CHANGED` | `/topic/lobby` | 로비의 방 목록 갱신 알림 |

