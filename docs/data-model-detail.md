# 데이터 모델 상세

[데이터 모델 설계](data-model.md)에서 이어지는 문서입니다.\
통계, 프로필, 운영 보조 도메인의 엔티티와 Enum 정의를 정리했습니다.

---

## 통계 도메인

### MatchupStat

방어 조합 vs 공격 조합의 승패 통계입니다. 배치 스케줄러가 전투 기록을 집계해서 갱신합니다.\
defense_combo + defense_pet + attack_combo + attack_pet 조합이 유니크 키입니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | matchup_stat_id |
| defenseCombo | `String` | NOT NULL | 방어 영웅 조합 (정렬, 콤마 구분) |
| defensePet | `String` | nullable | 방어 펫 |
| attackCombo | `String` | NOT NULL | 공격 영웅 조합 (정렬, 콤마 구분) |
| attackPet | `String` | nullable | 공격 펫 |
| wins | `int` | - | 승리 수 |
| losses | `int` | - | 패배 수 |
| totalGames | `int` | - | 총 경기 수 |
| winRate | `double` | - | 승률 (%) |

### MatchupSkillStat

같은 매치업 안에서 스킬 순서별 승패를 세분화한 통계입니다.\
matchup_stat_id + defense_skill_order + attack_skill_order 조합이 유니크 키입니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | matchup_skill_stat_id |
| matchupStat | `MatchupStat` | NOT NULL, FK | 상위 매치업 통계 |
| defenseSkillOrder | `String(200)` | nullable | 방어 스킬 순서 |
| attackSkillOrder | `String(200)` | nullable | 공격 스킬 순서 |
| wins / losses / totalGames / winRate | | | MatchupStat과 동일 구조 |

### MatchupComment

매치업 통계에 달리는 사용자 댓글입니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | matchup_comment_id |
| matchupStat | `MatchupStat` | NOT NULL, FK | 대상 매치업 |
| member | `Member` | NOT NULL, FK | 작성자 |
| content | `String(500)` | NOT NULL | 댓글 내용 |
| likeCount | `int` | - | 좋아요 수 |

### MatchupCommentLike

댓글 좋아요입니다. 댓글 + 회원 조합이 유니크로, 중복 좋아요를 방지합니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | - |
| comment | `MatchupComment` | NOT NULL, FK | 대상 댓글 |
| member | `Member` | NOT NULL, FK | 좋아요 누른 회원 |

### GameProfile

회원의 게임 프로필입니다. member_id를 PK로 사용하는 1:1 구조입니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| memberId | `Long` | PK | Member ID와 동일 |
| level | `Integer` | nullable | 게임 레벨 |

### GameRank

회원의 게임 내 랭크 정보입니다. 모드별로 여러 개가 있을 수 있습니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | game_rank_id |
| member | `Member` | NOT NULL, FK | 회원 |
| mode | `String(20)` | NOT NULL | 게임 모드 |
| tier | `String(30)` | NOT NULL | 티어 |
| rankPosition | `int` | NOT NULL | 순위 |
| winRate | `Double` | nullable | 승률 |
| score | `Integer` | nullable | 점수 |
| topPercent | `Double` | nullable | 상위 % |

### RankVerification

랭크 인증 요청입니다. 사용자가 스크린샷을 올리면 관리자가 승인/거절합니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | rank_verification_id |
| memberId | `Long` | NOT NULL | 요청 회원 ID |
| memberNickname | `String(30)` | NOT NULL | 요청 시점 닉네임 |
| imageUrl | `String` | NOT NULL | 인증 스크린샷 URL |
| memo | `String(500)` | nullable | 요청 메모 |
| status | `RankVerificationStatus` | NOT NULL | PENDING / APPROVED / REJECTED |
| rejectReason | `String(200)` | nullable | 거절 사유 |

---

## 길드전 데이터 도메인

### Hero

게임 내 영웅 마스터 데이터입니다. 이름이 유니크 키입니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | hero_id |
| name | `String` | NOT NULL, UNIQUE | 영웅 이름 |
| imageUrl | `String` | nullable | 영웅 이미지 URL |
| skill1ImageUrl | `String` | nullable | 스킬1 이미지 URL |
| skill2ImageUrl | `String` | nullable | 스킬2 이미지 URL |
| type | `HeroType` | nullable | ATTACK / DEFENSE / MAGIC / SUPPORT / ALL_ROUND |

### Pet

게임 내 펫 마스터 데이터입니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | pet_id |
| name | `String` | NOT NULL, UNIQUE | 펫 이름 |
| imageUrl | `String` | nullable | 펫 이미지 URL |

---

## 운영/관리 도메인

### Notice

공지사항입니다. 고정(pinned) 여부를 지원하고, 이미지는 콤마로 구분해서 저장합니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | - |
| title | `String(200)` | NOT NULL | 제목 |
| content | `TEXT` | NOT NULL | 내용 |
| pinned | `boolean` | NOT NULL | 고정 여부 |
| imageUrls | `TEXT` | nullable | 이미지 URL 목록 (콤마 구분) |

### Inquiry

1:1 문의입니다. 관리자가 답변하면 상태가 ANSWERED로 바뀝니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | - |
| memberId | `Long` | NOT NULL | 문의자 ID |
| memberNickname | `String(30)` | NOT NULL | 문의 시점 닉네임 |
| category | `String(20)` | NOT NULL | 문의 카테고리 |
| title | `String(200)` | NOT NULL | 제목 |
| content | `TEXT` | NOT NULL | 내용 |
| status | `InquiryStatus` | NOT NULL | OPEN / ANSWERED |
| answer | `TEXT` | nullable | 관리자 답변 |
| imageUrls | `TEXT` | nullable | 첨부 이미지 URL (콤마 구분) |

### Coupon

쿠폰 정보입니다. 코드가 유니크 키이고, 활성/비활성 상태를 관리합니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | - |
| code | `String(100)` | NOT NULL, UNIQUE | 쿠폰 코드 |
| name | `String(120)` | NOT NULL | 쿠폰 이름 |
| rewardSummary | `String(500)` | nullable | 보상 요약 |
| active | `boolean` | NOT NULL | 활성 여부 |

### Preset

사용자가 저장한 방어 진형 프리셋입니다. 데이터는 JSON 형태로 저장됩니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | preset_id |
| member | `Member` | NOT NULL, FK | 소유자 |
| name | `String(50)` | NOT NULL | 프리셋 이름 |
| data | `JSON` | nullable | 진형 데이터 |

### PushSubscription

FCM 푸시 알림 구독 정보입니다. 회원 + 토큰 조합이 유니크입니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | - |
| member | `Member` | NOT NULL, FK | 구독 회원 |
| fcmToken | `String(512)` | NOT NULL | FCM 토큰 |
| deviceType | `String(10)` | NOT NULL | 기기 타입 |

### AccessDailyStat

일별 접속 통계입니다. 관리자 대시보드에서 접속 추이를 확인할 때 사용합니다.\
날짜 + 접속 타입 조합이 유니크입니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | - |
| accessDate | `LocalDate` | NOT NULL | 접속 날짜 |
| accessType | `AccessType` | NOT NULL | STANDALONE / BROWSER / IN_APP_BROWSER |
| count | `int` | NOT NULL | 접속 횟수 |

### MemberDeleted / OAuthDeleted

회원 탈퇴 시 30일간 보관하는 백업 데이터입니다. 복구 요청 시 이 데이터로 계정을 되살립니다.

| 엔티티 | 주요 필드 | 설명 |
|--------|----------|------|
| MemberDeleted | id, nickname, email, deletedAt, expireAt | 탈퇴 회원 백업 (30일 보관) |
| OAuthDeleted | id, memberId, provider, oauthId | 탈퇴 회원 OAuth 백업 |

---

## Enum 정의

비즈니스 로직에서 사용하는 Enum 값들을 정리했습니다.

### 회원/인증

| Enum | 값 | 설명 |
|------|-----|------|
| Role | `GUEST` | 비회원 |
| | `MEMBER` | 일반 회원 |
| | `ADMIN` | 관리자 |
| OAuthProvider | `KAKAO` | 카카오 OAuth |
| | `GOOGLE` | 구글 OAuth |

### 방/전투

| Enum | 값 | 설명 |
|------|-----|------|
| RoomStatus | `ACTIVE` | 진행 중 (데이터 수정 가능) |
| | `CLOSED` | 종료됨 (1시간 버퍼 동안 수정 가능) |
| | `LOCKED` | 통계 반영 완료 (수정 불가) |
| RoomRole | `OWNER` | 방장 (방 관리, 역할 변경, 전투 편집) |
| | `MANAGER` | 부방장 (전투 편집) |
| | `USER` | 일반 멤버 |
| | `OBSERVER` | 옵저버 |
| RoomMemberStatus | `ACTIVE` | 활성 (방에 접속 중) |
| | `OFFLINE` | 오프라인 (일시 끊김, 참여는 유지) |
| | `KICKED` | 강퇴됨 (참여 불가) |
| CastleSide | `ALLY` | 아군 |
| | `ENEMY` | 적군 |
| CastleType | `OUTER` | 외성 (최대 방어 10명, 5개, 20점) |
| | `INNER` | 내성 (최대 방어 15명, 3개, 35점) |
| | `MAIN` | 본성 (최대 방어 20명, 1개, 50점) |
| Formation | `BASIC` | 기본 진형 |
| | `BALANCE` | 밸런스 진형 |
| | `ATTACK` | 공격 진형 |
| | `PROTECT` | 보호 진형 |
| BattleResult | `WIN` | 공격 성공 |
| | `LOSE` | 공격 실패 |
| BattleRecordStatus | `PENDING` | 집계 대기 중 |
| | `ACTIVE` | 집계 완료 |
| | `SKIPPED` | 불완전한 데이터로 집계 제외 |
| | `DISABLED` | 통계에서 제외 |

### 통계/운영

| Enum | 값 | 설명 |
|------|-----|------|
| HeroType | `ATTACK` / `DEFENSE` / `MAGIC` / `SUPPORT` / `ALL_ROUND` | 영웅 타입 |
| RankVerificationStatus | `PENDING` / `APPROVED` / `REJECTED` | 랭크 인증 상태 |
| InquiryStatus | `OPEN` / `ANSWERED` | 문의 상태 |
| AccessType | `STANDALONE` / `BROWSER` / `IN_APP_BROWSER` | 접속 타입 |
| CouponRedeemStatus | `SUCCESS` / `ALREADY_USED` / `FAILED` | 쿠폰 사용 결과 |

---

## 관련 문서

- [핵심 비즈니스 ERD](assets/erd-business-core.png)
- [운영/관리 ERD](assets/erd-admin-operation.png)
- [길드전 기록에서 통계 집계까지](scenarios/guild-war-to-stats.md)
- [테스트 전략](testing.md)
