# 공격 통계 조회 API

GET /stats/matchups/attacks

특정 방어 조합을 공격한 기록들을 조회합니다.

---

## [설명]

특정 `defenseCombo + defensePet`에 해당하는 개별 MatchupStat 레코드를 반환합니다.
각 레코드는 해당 방어 조합을 공격한 공격 조합의 승/패 기록입니다.
총 게임수 내림차순으로 정렬됩니다.

---

## [Request]

```
GET /stats/matchups/attacks?defenseCombo=아멜리아,카구라,파이&defensePet=멜키르
```

### 파라미터

| 파라미터 | 필수 | 타입 | 설명 |
|---------|------|------|------|
| defenseCombo | O | String | 방어 조합 |
| defensePet | X | String | 방어 펫 |

인증 없이 접근 가능합니다 (permitAll).

---

## [성공 응답]

```json
HTTP 200
{
  "success": true,
  "data": [
    {
      "id": 1,
      "defenseCombo": "아멜리아,카구라,파이",
      "defensePet": "멜키르",
      "attackCombo": "바네사,유신,카일",
      "attackPet": "이린",
      "wins": 12,
      "losses": 8,
      "totalGames": 20,
      "winRate": 60.0
    }
  ]
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | MatchupStat ID |
| defenseCombo | String | 방어 조합 |
| defensePet | String | 방어 펫 |
| attackCombo | String | 공격 조합 |
| attackPet | String | 공격 펫 |
| wins | int | 승수 |
| losses | int | 패수 |
| totalGames | int | 총 게임 수 |
| winRate | double | 승률 (%) |
