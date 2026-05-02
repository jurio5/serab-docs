# 전체 문의 목록 조회 API (관리자)

GET /admin/inquiries

관리자가 전체 문의를 키워드 검색 + 페이징으로 조회합니다.

---

## [Request]

```
GET /admin/inquiries?keyword=로그인&page=0&size=10
Cookie: access_token={JWT}
```

### 파라미터

| 파라미터 | 필수 | 타입 | 기본값 | 설명 |
|---------|------|------|--------|------|
| keyword | X | String | null | 제목 또는 닉네임 검색 |
| page | X | int | 0 | 페이지 번호 |
| size | X | int | 10 | 페이지 크기 |

관리자 권한이 필요합니다 (ADMIN).

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "memberId": 1,
        "memberNickname": "테스터",
        "category": "버그 신고",
        "title": "로그인 오류",
        "content": "내용",
        "status": "OPEN",
        "answer": null,
        "createdAt": "2026-03-04T12:00:00",
        "updatedAt": "2026-03-04T12:00:00"
      }
    ],
    "totalElements": 1,
    "totalPages": 1
  }
}
```

---

## [내부 처리 흐름]

1. keyword가 있으면 title 또는 memberNickname에 대해 LIKE 검색
2. Page 기반 페이지네이션 적용
3. InquiryResponse.from()으로 변환
