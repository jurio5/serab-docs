# 개인 통계 조회 API

GET /stats/my

인증된 사용자의 개인 전적 통계를 조회합니다.

---

## [설명]

로그인한 사용자의 BattleRecord를 기반으로 승/패를 집계하고,
인증된 게임 랭크와 프로필 정보를 함께 반환합니다.

### 승/패 판정 기준

| Castle Side | BattleResult | 개인 관점 |
|---|---|---|
| ALLY (아군 성) | WIN (공격 성공) | **패배** (방어 뚫림) |
| ALLY (아군 성) | LOSE (공격 실패) | **승리** (방어 성공) |
| ENEMY (적 성) | WIN (공격 성공) | **승리** (공격 성공) |
| ENEMY (적 성) | LOSE (공격 실패) | **패배** (공격 실패) |

---

## [Request]

```
GET /stats/my
Headers:
  Cookie: ACCESS_TOKEN=...
```

### 인증
필수 (로그인 필요)

---

## [성공 응답]

```json
HTTP 200
{
  "status": 200,
  "data": {
    "nickname": "테스터",
    "wins": 9,
    "losses": 6,
    "winRate": 60.0,
    "totalRecords": 15,
    "ranks": [
      {
        "mode": "결투장",
        "tier": "챌린저",
        "rank": 93,
        "winRate": 15.0,
        "score": 3800,
        "topPercent": 2.0
      }
    ],
    "ranksUpdatedAt": "2026-03-27T11:30:00",
    "level": 150
  }
}
```

### 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| nickname | String | 사용자 닉네임 |
| wins | long | 승리 횟수 |
| losses | long | 패배 횟수 |
| winRate | double | 승률 (%) |
| totalRecords | long | 총 전적 수 |
| ranks | GameRankResponse[] | 인증된 게임 랭크 목록 |
| ranksUpdatedAt | LocalDateTime | 랭크 마지막 갱신 일시 (null 가능) |
| level | Integer | 게임 레벨 (null 가능) |

### GameRankResponse 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| mode | String | 게임 모드 (예: 결투장, 총력전) |
| tier | String | 티어 (예: 챌린저, 다이아몬드) |
| rank | int | 순위 |
| winRate | Double | 승률 (null 가능) |
| score | Integer | 점수 (null 가능) |
| topPercent | Double | 상위 % (null 가능) |

---

## [내부 처리 흐름]

1. RequestContext에서 인증된 사용자 정보 추출
2. BattleRecord에서 `castle.side` + `result` 기준으로 GROUP BY 집계
3. CastleSide에 따라 승/패 관점 변환 (아군 성은 결과 반전)
4. GameRank, GameProfile에서 랭크 및 레벨 정보 조회
5. 통합 응답 반환
