# 관리자 공지사항 생성

POST /admin/notices

---

## [설명]

새 공지사항을 생성합니다.\
이미지 첨부 시 스토리지에 업로드 후 URL이 저장됩니다.

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...
Content-Type: multipart/form-data

Form Data:
title        (필수) 공지 제목 (최대 200자)
content      (필수) 공지 내용
pinned       (필수) 고정 여부 (true/false)
image        (선택) 첨부 이미지 파일
```

## [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": {
    "id": 3,
    "title": "새 공지사항",
    "content": "새 공지 내용입니다.",
    "pinned": false,
    "imageUrl": "https://cdn.example.com/notices/uuid.webp",
    "createdAt": "2026-03-01T12:00:00"
  }
}
```

---

## [실패 응답]

| ErrorCode | HTTP Status | 설명 |
|-----------|-------------|------|
| UNAUTHORIZED | 401 | 로그인되지 않음 |
| ACCESS_DENIED | 403 | ROLE_ADMIN이 아님 |
| ADMIN_GATE_REQUIRED | 403 | admin_gate 쿠키 없음 |
| IMAGE_UPLOAD_FAILED | 500 | 이미지 업로드 실패 |

---

## [내부 처리 흐름]

```
JWT 인증 → ROLE_ADMIN 인가 → SuspensionGateFilter → AdminGateFilter(쿠키 검증)
    ↓
AdminController.createNotice() → NoticeService.create()
    ↓
이미지 존재 시: StorageService.upload("notices/{UUID}.webp", ...)
    ↓
Notice.create(title, content, pinned, imageUrl) → Repository.save()
```
