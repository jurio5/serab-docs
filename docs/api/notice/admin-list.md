# 관리자 공지사항 목록 조회

GET /admin/notices

---

## [설명]

전체 공지사항 목록을 페이징으로 조회합니다.\
키워드 검색 시 제목에 대해 LIKE 검색을 수행합니다.\
고정(pinned) 공지가 최상단에, 나머지는 작성일 내림차순으로 정렬됩니다.

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...; admin_gate=...

Query Parameters:
keyword  (선택) 검색 키워드 (제목)
page     (선택, 기본값: 0)
size     (선택, 기본값: 20)
```

## [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": {
    "content": [
      {
        "id": 1,
        "title": "서버 점검 안내",
        "content": "2월 28일 새벽 2시~4시 서버 점검이 진행됩니다.",
        "pinned": true,
        "imageUrl": "https://cdn.example.com/notices/abc123.webp",
        "createdAt": "2026-02-28T10:00:00"
      }
    ],
    "totalElements": 15,
    "totalPages": 1,
    "number": 0,
    "size": 20,
    "first": true,
    "last": true
  }
}
```

## 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 공지사항 ID |
| title | String | 제목 |
| content | String | 내용 |
| pinned | boolean | 고정 여부 |
| imageUrl | String | 첨부 이미지 URL (nullable) |
| createdAt | LocalDateTime | 작성 일시 |

---

## [내부 처리 흐름]

```
JWT 인증 → ROLE_ADMIN 인가 → SuspensionGateFilter → AdminGateFilter(쿠키 검증)
    ↓
AdminController.getNotices() → NoticeService.getNotices()
    ↓
NoticeRepository.findAllWithSearch(keyword, pageable)
    ↓
keyword가 null이면 전체 조회, 있으면 제목 LIKE 검색
pinned DESC → createDate DESC 정렬
```
