# 좋아요 토글 API

POST /stats/comments/{commentId}/like

댓글에 좋아요를 토글합니다.

---

## [설명]

로그인한 사용자가 댓글에 좋아요를 누르거나 취소합니다.\
이미 좋아요를 누른 상태에서 다시 호출하면 좋아요가 취소됩니다.

---

## [Request]

```
POST /stats/comments/1/like

Headers:
Cookie: ACCESS_TOKEN=...
```

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": true   // true: 좋아요 추가, false: 좋아요 취소
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

1. 현재 사용자 + 댓글 ID로 기존 좋아요 조회
2. 있으면 → 삭제 + likeCount 감소 → `false` 반환
3. 없으면 → 생성 + likeCount 증가 → `true` 반환

---

## [동시성 처리]

| 관점 | 방식 | 설명 |
|------|------|------|
| 트랜잭션 | 클래스 레벨 `@Transactional` | insert/delete + likeCount 갱신이 하나의 트랜잭션으로 묶여 중간 실패 시 전체 롤백 |
| 중복 좋아요 방지 | UniqueConstraint + `DataIntegrityViolationException` catch | DB 유니크 제약으로 중복 insert 차단, 위반 시 `MATCHUP_LIKE_ALREADY_EXISTS` (409) 응답 |
| 삭제 안전성 | `deleteByCommentIdAndMemberId` 쿼리 기반 삭제 | 실제 삭제된 row 수(`deleted > 0`)를 확인한 후에만 `likeCount` 감소 |
| 원자적 카운트 갱신 | `@Modifying` JPQL 쿼리 | `UPDATE SET likeCount = likeCount ± 1`로 메모리 수정 없이 DB에서 원자적 갱신 |
