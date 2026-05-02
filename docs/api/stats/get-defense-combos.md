# 방어 조합 목록 조회 API

GET /stats/matchups

방어 조합별 집계 통계를 페이지 단위로 조회합니다.

---

## [설명]

MatchupStat 테이블을 `defenseCombo + defensePet` 기준으로 GROUP BY 하여
총 게임수, 승수, 패수, 평균 승률을 집계한 결과를 반환합니다.

영웅 이름으로 검색이 가능하며, Slice 기반 페이지네이션을 지원합니다.

---

## [Request]

```
GET /stats/matchups?hero=아멜리아,파이&page=0
```

### 파라미터

| 파라미터 | 필수 | 타입 | 기본값 | 설명 |
|---------|------|------|--------|------|
| hero | X | String | null | 영웅 이름 검색 (쉼표 구분으로 다중 AND 검색 가능) |
| page | X | int | 0 | 페이지 번호 (0부터) |

인증 없이 접근 가능합니다 (permitAll).

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": {
    "content": [
      {
        "defenseCombo": "아멜리아,카구라,파이",
        "defensePet": "멜키르",
        "totalGames": 106,
        "totalWins": 55,
        "totalLosses": 51,
        "overallWinRate": 51.89
      }
    ],
    "hasNext": true
  }
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| defenseCombo | String | 방어 조합 (쉼표 구분) |
| defensePet | String | 방어 펫 |
| totalGames | long | 총 게임 수 (집계) |
| totalWins | long | 총 승수 (집계) |
| totalLosses | long | 총 패수 (집계) |
| overallWinRate | double | 평균 승률 (%) |
| hasNext | boolean | 다음 페이지 존재 여부 |

---

## [내부 처리 흐름]

1. hero 파라미터를 쉼표로 split하여 영웅 리스트 생성 (예: `"아멜리아,파이"` → `["아멜리아", "파이"]`)
2. QueryDSL 동적 쿼리로 각 영웅에 대해 `defenseCombo LIKE '%영웅%'` AND 조건 생성
3. GROUP BY + SUM 집계 후 Slice(limit+1) 방식으로 페이징 처리
