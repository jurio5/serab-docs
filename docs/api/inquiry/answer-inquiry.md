# 문의 답변 API (관리자)

PATCH /admin/inquiries/{id}/answer

관리자가 문의에 답변합니다.

---

## [Request]

```
PATCH /admin/inquiries/1/answer
Cookie: access_token={JWT}
Content-Type: application/json

{
  "answer": "확인했습니다. 수정하겠습니다."
}
```

### 요청 필드

| 필드 | 필수 | 타입 | 설명 |
|------|------|------|------|
| answer | O | String | 답변 내용 |

관리자 권한이 필요합니다 (ADMIN).

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "id": 1,
    "memberId": 1,
    "memberNickname": "테스터",
    "category": "버그 신고",
    "title": "로그인 오류",
    "content": "내용",
    "status": "ANSWERED",
    "answer": "확인했습니다. 수정하겠습니다.",
    "createdAt": "2026-03-04T12:00:00",
    "updatedAt": "2026-03-04T13:00:00"
  }
}
```

---

## [에러 응답]

| 상태 | ErrorCode | 설명 |
|------|-----------|------|
| 404 | INQUIRY_NOT_FOUND | 해당 ID의 문의가 존재하지 않음 |

---

## [내부 처리 흐름]

1. id로 Inquiry 조회 (없으면 INQUIRY_NOT_FOUND 예외)
2. inquiry.answer() 호출 → answer 저장, 상태 ANSWERED로 변경
3. InquiryResponse.from()으로 변환
