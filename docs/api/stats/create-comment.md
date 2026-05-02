# 댓글 작성 API

POST /stats/matchups/{statId}/comments

특정 매치업 통계에 댓글을 작성합니다.

---

## [설명]

로그인한 사용자가 특정 MatchupStat에 댓글을 작성합니다.

---

## [Request]

```json
POST /stats/matchups/1/comments

Headers:
Cookie: ACCESS_TOKEN=...

Body:
{
  "content": "이 조합 카운터가 확실합니다"
}
```

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "id": 1,
    "nickname": "유저1",
    "memberId": 10,
    "content": "이 조합 카운터가 확실합니다",
    "likeCount": 0,
    "liked": false,
    "createdAt": "2026-02-15T12:00:00"
  }
}
```

---

## [실패 응답]

| ErrorCode | 설명 |
|-----------|------|
| MATCHUP_STAT_NOT_FOUND | 존재하지 않는 매치업 통계 ID |
| UNAUTHORIZED | 인증되지 않은 사용자 |
