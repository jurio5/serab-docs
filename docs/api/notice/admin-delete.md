# 관리자 공지사항 삭제

DELETE /admin/notices/{id}

---

## [설명]

특정 공지사항을 삭제합니다.\
이미지가 첨부된 공지사항의 경우 스토리지의 이미지도 함께 삭제됩니다.

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Path Parameters:
id  공지사항 ID
```

## [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": null
}
```

---

## [실패 응답]

| ErrorCode | HTTP Status | 설명 |
|-----------|-------------|------|
| UNAUTHORIZED | 401 | 로그인되지 않음 |
| ACCESS_DENIED | 403 | ROLE_ADMIN이 아님 |
| ADMIN_GATE_REQUIRED | 403 | admin_gate 쿠키 없음 |
| NOTICE_NOT_FOUND | 404 | 존재하지 않는 공지사항 ID |

---

## [내부 처리 흐름]

```
JWT 인증 → ROLE_ADMIN 인가 → SuspensionGateFilter → AdminGateFilter(쿠키 검증)
    ↓
AdminController.deleteNotice() → NoticeService.delete()
    ↓
이미지 URL이 존재하면: 스토리지에서 이미지 삭제
    ↓
NoticeRepository.delete(notice)
```
