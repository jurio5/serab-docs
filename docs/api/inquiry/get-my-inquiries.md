# 내 문의 목록 조회 API

GET /inquiries/my

로그인한 회원의 문의 목록을 조회합니다.

---

## [Request]

```
GET /inquiries/my
Cookie: access_token={JWT}
```

인증이 필요합니다.

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": [
    {
      "id": 1,
      "memberId": 1,
      "memberNickname": "테스터",
      "category": "버그 신고",
      "title": "로그인 오류",
      "content": "내용",
      "status": "ANSWERED",
      "answer": "확인했습니다.",
      "createdAt": "2026-03-04T12:00:00",
      "updatedAt": "2026-03-04T13:00:00"
    }
  ]
}
```

---

## [내부 처리 흐름]

1. RequestContext에서 인증된 회원 ID 추출
2. memberId 기준으로 문의 조회 (createDate DESC 정렬)
3. InquiryResponse.from()으로 변환
