# 매치업 상세 조회 API

GET /stats/matchups/{statId}/details

특정 매치업 통계의 실제 전투 기록 상세를 조회합니다.

---

## [설명]

MatchupStat의 방어/공격 조합과 펫 정보를 기반으로 BattleRecord에서 정확히 일치하는
최근 전투 기록 5건을 조회합니다.

QueryDSL을 사용하여 DB 레벨에서 직접 필터링하며, combo 컬럼 + 펫 조건으로 매칭합니다.

---

## [Request]

```
GET /stats/matchups/1/details
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
      "defenseFormation": "기본 진형",
      "defensePet": "멜페로",
      "defenseHeroes": [
        { "heroName": "카구라", "position": 1 },
        { "heroName": "카일", "position": 2 },
        { "heroName": "파이", "position": 3 }
      ],
      "attackFormation": "공격 진형",
      "attackPet": "이린",
      "attackHeroes": [
        { "heroName": "녹스", "position": 1 },
        { "heroName": "로지", "position": 2 },
        { "heroName": "아라곤", "position": 3 }
      ],
      "result": "WIN",
      "battleDate": "2026-03-04T12:00:00"
    }
  ]
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| defenseFormation | String | 방어 진형 (null 가능) |
| defensePet | String | 방어 펫 (null 가능) |
| defenseHeroes | HeroInfo[] | 방어 영웅 목록 (heroName, position) |
| attackFormation | String | 공격 진형 (null 가능) |
| attackPet | String | 공격 펫 (null 가능) |
| attackHeroes | HeroInfo[] | 공격 영웅 목록 (heroName, position) |
| result | String | 전투 결과 (WIN / LOSE) |
| battleDate | LocalDateTime | 전투 일시 |

---

## [에러 응답]

| 상태 | ErrorCode | 설명 |
|------|-----------|------|
| 404 | MATCHUP_STAT_NOT_FOUND | 해당 statId의 통계가 존재하지 않음 |

---

## [내부 처리 흐름]

1. statId로 MatchupStat 조회
2. stat의 defenseCombo, attackCombo, defensePet, attackPet 추출
3. QueryDSL로 BattleRecord 조회 (status=ACTIVE, combo 일치, pet 일치)
4. createDate DESC 정렬, LIMIT 5
5. MatchupDetailResponse.from()으로 변환
