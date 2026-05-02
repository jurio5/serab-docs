# 방 목록 조회 API

GET /rooms

상태별 방 목록을 조회하는 API입니다.

---

## 설명

지정한 상태의 방 목록을 조회합니다.  
상태를 지정하지 않으면 기본값은 `ACTIVE`입니다.

길드전 탭에서는 `keyword`를 함께 전달해 방 제목 또는 방장 닉네임으로 검색할 수 있습니다.
대기방에서 현재 방장이 비어 있는 경우 `ownerNickname`은 `방장 없음`으로 내려갈 수 있습니다.

---

## Request

```http
GET /rooms?status=ACTIVE&keyword=길드장
Cookie: ACCESS_TOKEN=...
```

### Query Parameters

- `status`: 방 상태, 기본값 `ACTIVE`
  - `ACTIVE`: 진행 중
  - `CLOSED`: 종료
  - `LOCKED`: 잠금
- `keyword`: 검색 키워드, 선택
  - 방 제목 또는 방장 닉네임에 대해 `LIKE` 검색

---

## 성공 응답

```json
HTTP 200
{
  "code": "OK",
  "message": null,
  "data": [
    {
      "id": 1,
      "title": "3월 14일 길드전",
      "guildWarDate": "2026-03-14",
      "ownerNickname": "길드장",
      "status": "ACTIVE",
      "memberCount": 5,
      "hasPassword": false,
      "waitingMode": true,
      "createdAt": "2026-03-14T19:30:00"
    }
  ]
}
```

---

## 실패 응답

가능한 ErrorCode:

- `UNAUTHORIZED`

---

## 내부 처리 흐름

1. 클라이언트 요청 수신
2. `status` 파라미터 확인, 없으면 `ACTIVE`
3. `keyword` 존재 여부 확인
4. 키워드가 없으면 상태 기준 전체 조회
5. 키워드가 있으면 방 제목 또는 방장 닉네임 기준 검색
6. `RoomResponse` 리스트로 변환 후 반환

---

## 실시간 동기화

로비 화면은 아래 WebSocket 이벤트를 함께 사용합니다.

- `ROOM_CREATED` : 새 방 추가
- `ROOM_CLOSED` : 종료된 방 제거
- `ROOM_MEMBER_COUNT_CHANGED` : `memberCount` 갱신
- `ROOM_UPDATED` : `title`, `hasPassword`, `waitingMode`, `ownerNickname` 등 방 정보 갱신
