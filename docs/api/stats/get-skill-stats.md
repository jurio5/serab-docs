# 공격 스킬 순서 통계 조회 API

GET /stats/matchups/{statId}/skill-stats

특정 매치업에서 가장 많이 사용된 공격 스킬 순서 Top 5를 조회합니다.

---

## [설명]

`MatchupStat`에 연결된 `MatchupSkillStat` 중 총 게임수 내림차순, 승률 내림차순으로 정렬된
상위 5개 공격 스킬 순서를 반환합니다.

Spring Data JPA의 `Pageable`을 사용하여 DB 레벨에서 `LIMIT 5`로 처리됩니다 (`PageRequest.of(0, 5)`).

---

## [Request]

```
GET /stats/matchups/1/skill-stats
```

### 파라미터

| 파라미터 | 필수 | 타입 | 설명 |
|---------|------|------|------|
| statId | O | Long (Path) | 매치업 통계 ID |

인증 없이 접근 가능합니다 (permitAll).

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": [
    {
      "skillOrder": "파이2,카일2,카구라1",
      "wins": 8,
      "losses": 2,
      "totalGames": 10,
      "winRate": 80.0
    },
    {
      "skillOrder": "카구라1,파이2,카일2",
      "wins": 5,
      "losses": 5,
      "totalGames": 10,
      "winRate": 50.0
    }
  ]
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| skillOrder | String | 스킬 순서 ("영웅명+스킬번호" 쉼표 구분, 예: "파이2,카일2,카구라1") |
| wins | int | 승수 |
| losses | int | 패수 |
| totalGames | int | 총 게임 수 |
| winRate | double | 승률 (%) |

---

## [에러 응답]

| 상태 | ErrorCode | 설명 |
|------|-----------|------|
| 404 | MATCHUP_STAT_NOT_FOUND | 해당 statId의 통계가 존재하지 않음 |

---

## [스킬 순서 형식]

```
"파이2,카일2,카구라1"
 └─ 영웅명 + 스킬번호(1 or 2) 를 쉼표로 구분
 └─ 앞에서부터 순서(1번째 → 2번째 → 3번째) 의미
```

## [내부 처리 흐름]

1. statId로 MatchupStat 조회
2. MatchupSkillStat에서 해당 stat의 기록을 totalGames DESC, winRate DESC 정렬
3. `PageRequest.of(0, 5)`으로 DB 레벨에서 상위 5건만 조회
4. SkillStatResponse로 변환하여 반환
