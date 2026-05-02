# 방어 스킬 순서 트렌드 조회 API

GET /stats/matchups/{statId}/defense-skill-trend

특정 매치업에서 방어팀이 가장 많이 사용한 스킬 순서 Top 3를 조회합니다.

---

## [설명]

`MatchupStat`에 연결된 `MatchupSkillStat` 중 총 게임수 내림차순, 승률 내림차순으로 정렬된
상위 3개 방어 스킬 순서를 반환합니다.

방어팀 스킬 순서는 고정된 데이터이므로 공격 통계(Top 5)보다 적은 Top 3만 노출합니다.
Spring Data JPA의 `Pageable`을 사용하여 DB 레벨에서 `LIMIT 3`으로 처리됩니다 (`PageRequest.of(0, 3)`).

---

## [Request]

```
GET /stats/matchups/1/defense-skill-trend
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
      "skillOrder": "카구라1,파이2,카일2",
      "wins": 3,
      "losses": 7,
      "totalGames": 10,
      "winRate": 30.0
    },
    {
      "skillOrder": "파이2,카구라1,카일2",
      "wins": 2,
      "losses": 4,
      "totalGames": 6,
      "winRate": 33.3
    }
  ]
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| skillOrder | String | 스킬 순서 ("영웅명+스킬번호" 쉼표 구분) |
| wins | int | 방어팀 관점의 승수 (공격 실패 횟수) |
| losses | int | 방어팀 관점의 패수 (공격 성공 횟수) |
| totalGames | int | 총 게임 수 |
| winRate | double | 방어 승률 (%) |

---

## [에러 응답]

| 상태 | ErrorCode | 설명 |
|------|-----------|------|
| 404 | MATCHUP_STAT_NOT_FOUND | 해당 statId의 통계가 존재하지 않음 |

---

## [내부 처리 흐름]

1. statId로 MatchupStat 조회
2. MatchupSkillStat에서 해당 stat의 기록을 totalGames DESC, winRate DESC 정렬
3. `PageRequest.of(0, 3)`으로 DB 레벨에서 상위 3건만 조회
4. SkillStatResponse로 변환하여 반환
