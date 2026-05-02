# 댓글 목록 조회 API

GET /stats/matchups/{statId}/comments

특정 매치업 통계의 댓글을 조회합니다.

---

## [설명]

특정 MatchupStat에 달린 댓글 목록을 좋아요 수 내림차순으로 반환합니다.
현재 로그인한 사용자의 좋아요 여부(liked)도 함께 포함됩니다.

---

## [Request]

```
GET /stats/matchups/1/comments

Headers:
Cookie: ACCESS_TOKEN=...
```

인증 없이 접근 가능합니다 (permitAll).
단, 좋아요 여부(liked) 계산을 위해 로그인한 사용자 정보를 사용합니다.

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": [
    {
      "id": 1,
      "nickname": "유저1",
      "memberId": 10,
      "content": "이 조합 카운터가 확실합니다",
      "likeCount": 5,
      "liked": true,
      "createdAt": "2026-02-15T12:00:00"
    }
  ]
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 댓글 ID |
| nickname | String | 작성자 닉네임 |
| memberId | Long | 작성자 ID |
| content | String | 댓글 내용 |
| likeCount | int | 좋아요 수 |
| liked | boolean | 현재 사용자의 좋아요 여부 |
| createdAt | DateTime | 작성 일시 |
