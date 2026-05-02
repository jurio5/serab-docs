# 방어팀 랭킹 조회

인기 방어 영웅 TOP 5를 조회합니다.  
`MatchupStat` 테이블의 `defenseCombo`에서 개별 영웅별 출현 빈도를 계산하고,
가장 많이 사용된 영웅을 기준으로 관련 조합을 그룹핑하여 반환합니다.

---

## 요청

```http
GET /stats/defense-ranking
```

### Parameters
없음

---

## 응답

### 200 OK

```json
{
  "status": 200,
  "data": {
    "rankings": [
      {
        "coreHero": "라드그리드",
        "totalGames": 18,
        "totalWins": 10,
        "winRate": 55.6,
        "combos": [
          {
            "defenseCombo": "라드그리드,손오공,엘리시아",
            "totalGames": 12,
            "wins": 6,
            "winRate": 50.0
          },
          {
            "defenseCombo": "라드그리드,손오공,아델리아",
            "totalGames": 6,
            "wins": 4,
            "winRate": 66.7
          }
        ]
      }
    ],
    "lastUpdated": "2026-03-27T03:00:00"
  }
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `rankings` | `DefenseRankingResponse[]` | 코어 영웅별 랭킹 목록 |
| `lastUpdated` | `LocalDateTime` | 마지막 갱신 시각 |
| `rankings[].coreHero` | `String` | 코어 영웅 이름 (그룹 기준) |
| `rankings[].totalGames` | `long` | 이 영웅이 포함된 모든 조합의 총 전투 수 |
| `rankings[].totalWins` | `long` | 총 승리 수 |
| `rankings[].winRate` | `double` | 승률 (%) |
| `rankings[].combos` | `ComboDetailResponse[]` | 이 영웅이 포함된 개별 조합 목록 |
| `rankings[].combos[].defenseCombo` | `String` | 방어 영웅 조합 (콤마 구분) |
| `rankings[].combos[].totalGames` | `long` | 해당 조합의 총 전투 수 |
| `rankings[].combos[].wins` | `long` | 해당 조합의 승리 수 |
| `rankings[].combos[].winRate` | `double` | 해당 조합의 승률 (%) |

---

## 비고
- 최대 5개 코어 영웅을 반환합니다.
- 각 영웅별 `totalGames` 합산 기준 내림차순 정렬됩니다.
- 방어 펫은 무시하고 영웅 조합만으로 집계합니다.
- 동일 영웅이 여러 코어 영웅 그룹에 중복 등장할 수 있습니다.
- 데이터는 메모리 캐시로 관리되며, 통계 집계 스케줄러(화/목/일 03:00) 실행 시 갱신됩니다.

