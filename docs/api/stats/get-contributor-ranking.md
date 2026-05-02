# 기여자 랭킹 조회

주간 및 전체 기간의 기여자 랭킹을 조회합니다.  
기록, 댓글, 좋아요 3가지 카테고리로 각각 TOP 5를 반환합니다.

---

## 요청

```http
GET /stats/contributor-ranking
```

### Parameters
없음

### 인증
불필요 (비로그인 사용자도 접근 가능)

---

## 응답

### 200 OK

```json
{
  "status": 200,
  "data": {
    "records": {
      "weekly": [
        { "nickname": "유저A", "count": 24, "rank": 1 },
        { "nickname": "유저B", "count": 12, "rank": 2 }
      ],
      "allTime": [
        { "nickname": "유저A", "count": 156, "rank": 1 },
        { "nickname": "유저C", "count": 98, "rank": 2 }
      ]
    },
    "comments": {
      "weekly": [],
      "allTime": []
    },
    "likes": {
      "weekly": [],
      "allTime": []
    },
    "lastUpdated": "2026-03-27T03:00:00"
  }
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `records` | `CategoryRanking` | 전투 기록 기여 랭킹 |
| `comments` | `CategoryRanking` | 댓글 작성 랭킹 |
| `likes` | `CategoryRanking` | 좋아요 받은 수 랭킹 |
| `lastUpdated` | `LocalDateTime` | 마지막 갱신 시각 |
| `*.weekly` | `ContributorRankingResponse[]` | 주간 랭킹 (월요일 00:00 기준) |
| `*.allTime` | `ContributorRankingResponse[]` | 전체 기간 랭킹 |
| `*.weekly[].nickname` | `String` | 사용자 닉네임 |
| `*.weekly[].count` | `long` | 기여 수 |
| `*.weekly[].rank` | `int` | 순위 (1부터) |

---

## 비고
- 각 카테고리별 최대 5명까지 반환합니다.
- 데이터는 메모리 캐시로 관리되며, 통계 집계 스케줄러(화/목/일 03:00) 실행 시 갱신됩니다.
- 기록 랭킹: `BattleRecord`의 `recordedBy` 기준, ACTIVE 상태만 집계
- 댓글 랭킹: `MatchupComment` 작성 수 기준
- 좋아요 랭킹: 댓글 작성자가 받은 `likeCount` 합산 기준
