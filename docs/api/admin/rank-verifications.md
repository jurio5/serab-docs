# 관리자 랭크 인증 관리 API

관리자가 랭크 인증 요청을 조회, 승인, 반려, 삭제합니다.

---

## 인증 요청 목록 조회

```
GET /admin/rank-verifications?keyword=&page=0&size=20&sort=createDate,desc
```

### 파라미터

| 파라미터 | 필수 | 타입 | 기본값 | 설명 |
|---------|------|------|--------|------|
| keyword | ❌ | String | null | 닉네임 검색 |
| page | ❌ | int | 0 | 페이지 번호 |
| size | ❌ | int | 20 | 페이지 크기 |
| sort | ❌ | String | createDate,desc | 정렬 기준 |

### 성공 응답

```json
HTTP 200
{
  "status": 200,
  "data": {
    "content": [
      {
        "id": 1,
        "memberId": 5,
        "memberNickname": "테스터",
        "imageUrl": "https://r2.example.com/rank-verifications/xxx.webp",
        "memo": "결투장 시즌10",
        "status": "PENDING",
        "rejectReason": null,
        "createdAt": "2026-03-27T11:00:00"
      }
    ],
    "totalElements": 1,
    "totalPages": 1,
    "currentPage": 0
  }
}
```

---

## 인증 승인

```
POST /admin/rank-verifications/{verificationId}/approve
Content-Type: application/json
```

### 요청 바디

```json
{
  "level": 150
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|----|------|
| level | Integer | X  | 플레이어 레벨 (GameProfile에 저장) |

### 처리 흐름

1. 인증 요청 상태를 APPROVED로 변경
2. `level`이 제공된 경우 GameProfile upsert (생성 또는 갱신)
3. 랭크 데이터는 별도 `PUT /admin/members/{memberId}/game-ranks` API로 반영

### 성공 응답

```json
HTTP 200
{ "status": 200, "data": null }
```

---

## 인증 반려

```
POST /admin/rank-verifications/{verificationId}/reject
Content-Type: application/json
```

### 요청 바디

```json
{
  "reason": "스크린샷이 잘 보이지 않습니다"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|----|------|
| reason | String | O  | 반려 사유 |

### 성공 응답

```json
HTTP 200
{ "status": 200, "data": null }
```

---

## 인증 삭제

```
DELETE /admin/rank-verifications/{verificationId}
```

### 처리 흐름

1. R2에 업로드된 이미지 삭제
2. DB에서 인증 요청 레코드 삭제

### 성공 응답

```json
HTTP 200
{ "status": 200, "data": null }
```

---

## 비고
- 모든 API는 관리자 인증(ROLE_ADMIN + admin_gate 쿠키) 필요
- 승인 시 프론트 모달에서 입력한 랭크 정보는 `PUT /admin/members/{memberId}/game-ranks`로 별도 전송
- 삭제 시 R2 이미지도 함께 정리됨
