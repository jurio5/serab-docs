# 댓글 삭제 API

DELETE /stats/comments/{commentId}

댓글을 삭제합니다.

---

## [설명]

로그인한 사용자가 본인이 작성한 댓글을 삭제합니다.\
본인 댓글이 아닌 경우 삭제가 거부됩니다.

---

## [Request]

```
DELETE /stats/comments/1

Headers:
Cookie: ACCESS_TOKEN=...
```

---

## [성공 응답]

```json
HTTP 200
{
  "success": true
}
```

---

## [실패 응답]

| ErrorCode | 설명 |
|-----------|------|
| MATCHUP_COMMENT_NOT_FOUND | 존재하지 않는 댓글 ID |
| UNAUTHORIZED | 인증되지 않은 사용자 |

---

## [내부 처리 흐름]

1. 댓글 ID로 댓글 조회
2. 현재 사용자가 작성자인지 검증 (validateDeletableBy)
3. 검증 통과 시 삭제
