# 관리자 공지사항 수정

PUT /admin/notices/{id}

---

## [설명]

기존 공지사항을 수정합니다.\
이미지 교체, 이미지 삭제(removeImage), 텍스트만 수정 등을 지원합니다.

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...
Content-Type: multipart/form-data

Path Parameters:
id  공지사항 ID

Form Data:
title        (필수) 공지 제목 (최대 200자)
content      (필수) 공지 내용
pinned       (필수) 고정 여부 (true/false)
image        (선택) 새 첨부 이미지 파일
removeImage  (선택) 기존 이미지 삭제 여부 (true/false)
```

## [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": {
    "id": 1,
    "title": "수정된 제목",
    "content": "수정된 내용",
    "pinned": true,
    "imageUrl": "https://cdn.example.com/notices/new-uuid.webp",
    "createdAt": "2026-02-28T10:00:00"
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
| NOTICE_NOT_FOUND | 404 | 존재하지 않는 공지사항 ID |
| IMAGE_UPLOAD_FAILED | 500 | 이미지 업로드 실패 |

---

## [내부 처리 흐름]

```
JWT 인증 → ROLE_ADMIN 인가 → SuspensionGateFilter → AdminGateFilter(쿠키 검증)
    ↓
AdminController.updateNotice() → NoticeService.update()
    ↓
1. removeImage=true이면: 기존 이미지 스토리지에서 삭제 → notice.clearImage()
2. 새 이미지 존재 시: StorageService.upload() → 기존 이미지가 있으면 스토리지에서 삭제
3. notice.update(title, content, pinned, newImageUrl)
```

### [이미지 처리 케이스]

```
removeImage=true, image=null    → 기존 이미지 삭제, 이미지 없는 상태
removeImage=false, image=있음   → 기존 이미지 삭제 후 새 이미지 교체
removeImage=false, image=null   → 이미지 변경 없음
```
