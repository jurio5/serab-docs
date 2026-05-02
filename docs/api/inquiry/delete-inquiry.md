# 문의 삭제 API (관리자)

DELETE /admin/inquiries/{id}

관리자가 문의를 삭제합니다.

---

## [Request]

```
DELETE /admin/inquiries/1
Cookie: access_token={JWT}
```

관리자 권한이 필요합니다 (ADMIN).

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": null
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
2. inquiryRepository.delete() 호출
