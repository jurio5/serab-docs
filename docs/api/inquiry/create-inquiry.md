# 문의 작성 API

POST /inquiries

1:1 문의를 작성합니다.

---

## [Request]

```
POST /inquiries
Cookie: access_token={JWT}
Content-Type: application/json

{
  "category": "버그 신고",
  "title": "로그인 오류",
  "content": "로그인이 되지 않습니다."
}
```

### 요청 필드

| 필드 | 필수 | 타입 | 설명 |
|------|------|------|------|
| category | O | String | 문의 카테고리 (예: 버그 신고, 건의, 기타) |
| title | O | String | 문의 제목 (최대 200자) |
| content | O | String | 문의 내용 |

인증이 필요합니다.

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
    "content": "로그인이 되지 않습니다.",
    "status": "OPEN",
    "answer": null,
    "createdAt": "2026-03-04T12:00:00",
    "updatedAt": "2026-03-04T12:00:00"
  }
}
```

---

## [내부 처리 흐름]

1. RequestContext에서 인증된 회원 정보 추출
2. Inquiry 엔티티 생성 (상태: OPEN)
3. DB 저장 후 InquiryResponse.from()으로 변환
