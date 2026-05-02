# 최근 공지사항 조회

GET /notices

---

## [설명]

최근 공지사항 5개를 조회합니다.\
고정(pinned) 공지가 최상단에, 나머지는 작성일 내림차순으로 정렬됩니다.

## [Request]

```
Headers:
Cookie: ACCESS_TOKEN=...
```

별도의 파라미터 없이 호출합니다.

## [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": [
    {
      "id": 1,
      "title": "서버 점검 안내",
      "content": "2월 28일 새벽 2시~4시 서버 점검이 진행됩니다.",
      "pinned": true,
      "imageUrl": "https://cdn.example.com/notices/abc123.webp",
      "createdAt": "2026-02-28T10:00:00"
    },
    {
      "id": 2,
      "title": "업데이트 안내",
      "content": "새로운 영웅이 추가되었습니다.",
      "pinned": false,
      "imageUrl": null,
      "createdAt": "2026-02-27T15:30:00"
    }
  ]
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
JWT 인증 → NoticeController.getRecentNotices()
    ↓
NoticeService.getRecentNotices()
    ↓
NoticeRepository.findTop5ByOrderByPinnedDescCreateDateDesc()
    ↓
pinned DESC → createDate DESC 순으로 최대 5건 반환
```
