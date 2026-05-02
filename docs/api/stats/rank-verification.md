# 랭크 인증 요청 API

POST /stats/rank-verification

게임 랭크 인증 스크린샷을 제출합니다.

---

## [설명]

사용자가 게임 내 랭크 스크린샷을 업로드하여 인증 요청을 합니다.  
이미지는 R2에 저장되며, 관리자가 검토 후 승인/반려합니다.

---

## [Request]

```
POST /stats/rank-verification
Content-Type: multipart/form-data
Headers:
  Cookie: ACCESS_TOKEN=...
```

### 요청 필드

| 필드 | 타입 | 필수 | 설명 |
|------|------|----|------|
| image | MultipartFile | O  | 인증 스크린샷 이미지 |
| memo | String | X  | 추가 메모 (선택) |

### 인증
필수 (로그인 필요)

---

## [성공 응답]

```json
HTTP 200
{
  "status": 200,
  "data": {
    "id": 1,
    "memberId": 5,
    "memberNickname": "테스터",
    "imageUrl": "https://r2.example.com/rank-verifications/xxx.webp",
    "memo": "결투장 시즌10 기록",
    "status": "PENDING",
    "rejectReason": null,
    "createdAt": "2026-03-27T11:00:00"
  }
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 인증 요청 ID |
| memberId | Long | 요청자 회원 ID |
| memberNickname | String | 요청자 닉네임 |
| imageUrl | String | 업로드된 이미지 URL |
| memo | String | 메모 (null 가능) |
| status | String | 상태: PENDING, APPROVED, REJECTED |
| rejectReason | String | 반려 사유 (null 가능) |
| createdAt | LocalDateTime | 요청 일시 |

---

# 내 인증 요청 목록 조회

GET /stats/rank-verification/my

현재 사용자의 랭크 인증 요청 이력을 조회합니다.

---

## [Request]

```
GET /stats/rank-verification/my
Headers:
  Cookie: ACCESS_TOKEN=...
```

### 인증
필수 (로그인 필요)

---

## [성공 응답]

```json
HTTP 200
{
  "status": 200,
  "data": [
    {
      "id": 1,
      "memberId": 5,
      "memberNickname": "테스터",
      "imageUrl": "https://r2.example.com/rank-verifications/xxx.webp",
      "memo": null,
      "status": "APPROVED",
      "rejectReason": null,
      "createdAt": "2026-03-27T11:00:00"
    }
  ]
}
```

---

## [비고]
- 최신 순으로 정렬됩니다 (createDate DESC).
- 상태에 따라 프론트에서 뱃지 표시: PENDING(검토 중), APPROVED(승인), REJECTED(반려됨).
- 반려된 경우 `rejectReason` 필드에 사유가 포함됩니다.
